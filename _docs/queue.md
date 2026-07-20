---
layout: page
name: Queue
---

# Queue

UglyDuckling ships a small database-backed job queue for deferring work — sending emails, calling slow external APIs, generating reports — outside the request/response cycle. Jobs are plain PHP classes; a CLI tool (`ud-queue`) tracks and runs them, in the same spirit as `ud-migrate`.

There is no external broker (Redis, RabbitMQ, SQS, ...) — jobs are rows in a table, applied through the PDO connection your application already has. This keeps the moving parts small; if you outgrow it, the `QueueJob`/`QueueRepository`/`QueueWorker` classes are a natural boundary to swap for something bigger later.

---

## Job skeleton

A job is a plain class extending `QueueJob`. It holds its own data as constructor-set properties, and knows how to turn that data into a JSON-safe array and back — this is what gets stored in, and rebuilt from, the database, instead of relying on PHP's `serialize()`/`unserialize()` on arbitrary objects.

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Queue\QueueJob;

class SendWelcomeEmailJob extends QueueJob {

    public function __construct(private int $userId) {
    }

    public function handle(): void {
        // send the email, e.g. via a Mailer/DAO fetched however your app wires dependencies
    }

    public function toPayload(): array {
        return ['userId' => $this->userId];
    }

    public static function fromPayload(array $payload): static {
        return new static($payload['userId']);
    }

}
{% endhighlight %}

---

## Pushing a job

Jobs are queued from application code through `QueueRepository`, not from the CLI — the same way migration files are authored as code, not typed at the command line.

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Queue\QueueRepository;

$repository = new QueueRepository($pdo);

$repository->push('emails', SendWelcomeEmailJob::class, ['userId' => $userId]);
{% endhighlight %}

`push()` takes:

| Parameter | Meaning |
|---|---|
| `$queue` | Name of the queue, e.g. `'emails'`, `'reports'`. Workers only pull jobs from the queue they're told to work. |
| `$jobClass` | Must extend `QueueJob`; a class that doesn't throws `InvalidArgumentException` immediately, before anything is written. |
| `$payload` | The array returned by `toPayload()` when the job is later reconstructed. |
| `$delaySeconds` | *(default `0`)* Don't make the job available for reservation until this many seconds from now. |
| `$maxAttempts` | *(default `3`)* How many times to try before giving up and marking the job failed. |

---

## Connecting to the database

`ud-queue` uses the same convention as `ud-migrate`: it looks for a `migrations-config.php` file in the current directory — a plain PHP file that must `return` a `PDO` instance. See [Migrations » Connecting to the database]({{site.baseurl}}/docs/migrations#connecting-to-the-database) if you don't have one yet; both commands can share the same config file.

---

## Command reference

### `ud-queue work [config-file] [--queue=default] [--sleep=3] [--max-jobs=0]`

Runs the worker: reserves and runs jobs from `--queue` one at a time. With `--max-jobs=0` (the default) it runs forever, sleeping `--sleep` seconds between polls whenever the queue is empty — this is what you point `supervisor`/`systemd`/cron at in production. With `--max-jobs` set to a positive number, it stops once that many jobs have been processed, or as soon as the queue empties, whichever comes first — useful for a bounded run from a cron job.

{% highlight bash %}
ud-queue work --queue=emails
# Working queue 'emails' (forever)...
{% endhighlight %}

### `ud-queue status [config-file] [--queue=default]`

Reports how many jobs are pending and how many have permanently failed on a queue:

{% highlight bash %}
ud-queue status --queue=emails
# Queue 'emails': 3 pending, 1 failed
{% endhighlight %}

---

## How it tracks progress

A `queue_jobs` table is created automatically (on first `work` or `status` call) to hold pending, reserved, and failed jobs. You never need to create or touch this table yourself. A job's row is deleted once it completes successfully — only jobs still pending, or permanently failed, stick around.

---

## Retries and backoff

If `handle()` throws, the job is not lost. As long as its attempt count is below `max_attempts`:

* the job is released back onto the queue,
* its `attempts` counter increments,
* it becomes reservable again after an exponential backoff delay (`2^attempts` seconds, capped at 60).

Once `attempts` reaches `max_attempts`, the job is instead marked permanently failed: its row is kept (not deleted) with the error message and a `failed_at` timestamp, and it no longer counts as pending — it stops appearing to workers, and shows up under `failed` in `ud-queue status` for inspection.

---

## Running the worker without the CLI

`QueueWorker` can be used directly from PHP — useful for running a single pass as part of another script:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Queue\QueueRepository;
use Fabiom\UglyDuckling\Framework\Queue\QueueWorker;

$repository = new QueueRepository($pdo);
$repository->ensureTableExists();

$worker = new QueueWorker($repository);

$worker->processNextJob('emails'); // runs a single job, returns false if the queue was empty
$worker->work('emails', sleepSeconds: 3, maxJobs: 10); // runs up to 10 jobs, or until the queue empties
{% endhighlight %}
