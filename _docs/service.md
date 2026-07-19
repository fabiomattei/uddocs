---
layout: page
name: Service
---

# Service

Business logic that used to live directly inside `getRequest()`/`postRequest()` can be extracted into a **Service**: a plain PHP class, unrelated to the controller lifecycle, that receives its collaborators through the constructor and has no dependency on `PageStatus`, session state, or superglobals. This makes it possible to unit test the logic directly, without a controller, a session, or an HTTP request.

A Service implements the empty marker interface `Fabiom\UglyDuckling\Framework\Services\Service`. The interface carries no methods — it exists for discoverability (`instanceof Service`, IDE navigation) and to signal intent. The actual method name, arguments, and return type are up to each Service, since different operations naturally have different shapes.

Don't confuse `Service` with `BaseUseCase` (`Framework\UseCases`): `BaseUseCase` is coupled to `PageStatus` and a JSON use-case structure, and is part of the JSON-resource action system. `Service` is unrelated and intentionally has no dependency on the request lifecycle.

---

## Minimal example

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Services\Service;

class CreateBookService implements Service {

    public function __construct(
        private BookDao $bookDao
    ) {}

    public function execute(string $title, int $authorId): int {
        return $this->bookDao->insert([
            'title'     => $title,
            'author_id' => $authorId,
        ]);
    }

}
{% endhighlight %}

---

## Calling a Service from a controller

The controller stays responsible for validation, authorization, and rendering; it builds the Service, hands it validated input, and maps the result onto the view.

{% highlight php %}
public function postRequest() {
    $bookDao = new BookDao();
    $bookDao->setDBH($this->dbconnection->getDBH());

    $service = new CreateBookService($bookDao);
    $this->bookId = $service->execute(
        $this->postParameters['title'],
        $_SESSION['user_id']
    );

    $this->redirectToPreviousPage();
}
{% endhighlight %}

---

## Testing

Because a Service depends only on what is passed to its constructor, it can be tested directly, with a mocked DAO, without booting a controller or a session:

{% highlight php %}
public function testCreateBookServiceInsertsWithGivenAuthor(): void {
    $bookDao = $this->createMock(BookDao::class);
    $bookDao->expects($this->once())
        ->method('insert')
        ->with(['title' => 'Hamlet', 'author_id' => 42])
        ->willReturn(7);

    $service = new CreateBookService($bookDao);

    $this->assertSame(7, $service->execute('Hamlet', 42));
}
{% endhighlight %}

---

This pattern is new in UglyDuckling: only the marker interface exists so far, and using it is not required. It targets the cases described in <a href="{{site.baseurl}}/docs/controller">Controller</a> where business logic inside `getRequest()`/`postRequest()` has grown complex enough to need isolated testing.
