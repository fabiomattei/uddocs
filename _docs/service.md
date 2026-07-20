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

A password-change form is a typical case: the rule ("old password must match, new passwords must agree") has nothing to do with HTTP, but used to live inside `postRequest()` alongside session access and redirects. Pulled into a Service, failure paths become typed exceptions instead of inline flash messages:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Services\Service;

class ChangePasswordService implements Service {

    public function __construct(
        private UserDao $userDao
    ) {}

    /**
     * @throws OldPasswordMismatchException
     * @throws NewPasswordsDoNotMatchException
     */
    public function execute(int $userId, string $oldPassword, string $newPassword, string $retypeNewPassword): void {
        $user = $this->userDao->getById($userId);

        if (!$this->userDao->checkEmailAndPassword($user->usr_email, $oldPassword)) {
            throw new OldPasswordMismatchException();
        }

        if ($newPassword !== $retypeNewPassword) {
            throw new NewPasswordsDoNotMatchException();
        }

        $this->userDao->updatePassword($userId, $newPassword);
    }

}
{% endhighlight %}

---

## Calling a Service from a controller

The controller stays responsible for validation, authorization, and rendering; it builds the Service, hands it validated input, and maps each outcome (success or a specific exception) onto a flash message and a redirect.

{% highlight php %}
public function postRequest() {
    $this->userDao->setDBH($this->pageStatus->getDbconnection()->getDBH());

    $changePasswordService = new ChangePasswordService($this->userDao);

    try {
        $changePasswordService->execute(
            $_SESSION['user_id'],
            $this->postParameters[self::FIELD_OLD_PASSWORD],
            $this->postParameters[self::FIELD_NEW_PASSWORD],
            $this->postParameters[self::FIELD_RETYPE_NEW_PASSWORD]
        );
        $this->setSuccess('Password successfully updated');
        $this->redirectToSecondPreviousPage();
    } catch (OldPasswordMismatchException $e) {
        $this->setError('The old password field does not match with saved password');
        $this->redirectToPreviousPage();
    } catch (NewPasswordsDoNotMatchException $e) {
        $this->setError('The two new passwords do not match');
        $this->redirectToPreviousPage();
    }
}
{% endhighlight %}

---

## Testing

Because a Service depends only on what is passed to its constructor, it can be tested directly, with a mocked DAO, without booting a controller or a session:

{% highlight php %}
public function testUpdatesPasswordWhenOldPasswordMatchesAndNewPasswordsAgree() {
    $user = (object) ['usr_email' => 'jane@example.com'];

    $userDao = $this->createMock(UserDao::class);
    $userDao->method('getById')->with(42)->willReturn($user);
    $userDao->method('checkEmailAndPassword')->with('jane@example.com', 'oldpass')->willReturn(true);
    $userDao->expects($this->once())->method('updatePassword')->with(42, 'newpass');

    $service = new ChangePasswordService($userDao);
    $service->execute(42, 'oldpass', 'newpass', 'newpass');
}

public function testThrowsWhenOldPasswordDoesNotMatch() {
    $userDao = $this->createMock(UserDao::class);
    $userDao->method('getById')->willReturn((object) ['usr_email' => 'jane@example.com']);
    $userDao->method('checkEmailAndPassword')->willReturn(false);
    $userDao->expects($this->never())->method('updatePassword');

    $this->expectException(OldPasswordMismatchException::class);

    $service = new ChangePasswordService($userDao);
    $service->execute(42, 'wrongoldpass', 'newpass', 'newpass');
}
{% endhighlight %}

---

## What this does and does not make testable

Extracting a Service makes *that logic* testable — nothing else changes. The controller itself (`Controller` or `BaseController`) stays exactly as hard to unit test as before: it still reads `$_GET`/`$_POST`/`$_SESSION` directly, calls `header()`/`exit()` in its redirect helpers, and (for `BaseController`) renders by `extract()`-ing properties into a required view file. None of that is mockable without a real request/session.

That is by design, not an oversight: the goal is not to make the controller class testable, it's to leave as little logic in it as possible so there is little left worth testing there. A controller built this way should read as plumbing — validate, delegate to a Service, map the outcome to a flash message and a redirect — with the actual behavior covered by tests against the Service instead.

---

This pattern is new in UglyDuckling: only the marker interface exists so far, and using it is not required. It targets the cases described in <a href="{{site.baseurl}}/docs/controller">Controller</a> and <a href="{{site.baseurl}}/docs/component">Component</a> where business logic inside `getRequest()`/`postRequest()` (or `get_request()`/`post_request()`) has grown complex enough to need isolated testing.
