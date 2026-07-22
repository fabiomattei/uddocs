---
layout: page
name: Controller
---

# Controller

UglyDuckling covers 90% of use cases through JSON resources. When business logic becomes too complex for a JSON resource, you implement a controller.

A controller in UglyDuckling works similarly to a controller in any MVC framework. It implements `getRequest()` and `postRequest()` methods, validates and filters incoming parameters, queries the database, and exposes data to a view file.

When the logic inside `getRequest()`/`postRequest()` grows complex enough to need isolated testing, extract it into a <a href="{{site.baseurl}}/docs/service">Service</a> and keep the controller itself limited to validation, delegating to the Service, and mapping the outcome to a view or redirect.

---

## Controller skeleton

{% highlight php %}
class MyController extends BaseController {

    public function __construct() {
        parent::__construct();
        $this->classCompleteName = __CLASS__;
        $this->className         = 'MyController';
        $this->chapter           = 'Website';
        $this->templateFile      = 'websitetemplate';
        $this->viewFile          = 'src/Chapters/Website/Views/MyController';
        $this->controllerPointer = $this;
        $this->appTitle          = 'My website title';
    }

    public function check_authorization_get_request() {
        return true;
    }

    public $get_validation_rules = ['authorid' => 'required|integer'];
    public $get_filter_rules     = ['authorid' => 'trim'];

    public function getRequest() {
    }

    public function check_authorization_post_request() {
        return true;
    }

    public $post_validation_rules = ['title' => 'required|max_len,255'];
    public $post_filter_rules     = ['title' => 'trim|sanitize_string'];

    public function postRequest() {
    }

}
{% endhighlight %}

---

## Constructor

A controller must extend `BaseController`. The constructor sets several properties that the framework uses to route requests, load templates, and render views.

| Property | Description |
|---|---|
| `$this->classCompleteName` | Fully qualified class name (`__CLASS__`). Useful in shared view and template files to identify which controller was called. |
| `$this->className` | Short class name. Same use as above. |
| `$this->chapter` | The chapter (domain folder) the controller belongs to. Controllers live at `src/Chapters/{Chapter}/Controllers/`. |
| `$this->templateFile` | Name of the template file inside `src/Templates/` (without `.php`). |
| `$this->viewFile` | Path prefix for the view file. The framework appends `Get.php`, `Post.php`, `GetError.php`, or `PostError.php` depending on the request. |
| `$this->controllerPointer` | A reference to `$this`. Required for the template to access controller properties. |
| `$this->appTitle` | Application title, typically used in the `<title>` tag of the template. |

{% highlight php %}
public function __construct() {
    parent::__construct();
    $this->classCompleteName = __CLASS__;
    $this->className         = 'MyController';
    $this->chapter           = 'Website';
    $this->templateFile      = 'websitetemplate';
    $this->viewFile          = 'src/Chapters/Website/Views/MyController';
    $this->controllerPointer = $this;
    $this->appTitle          = 'My website title';
}
{% endhighlight %}

Each controller must also carry a `#[Route]` attribute mapping it to a URL slug:

{% highlight php %}
use Fabiom\UglyDuckling\Framework\Routing\Route;

#[Route(name: 'mycontroller', slug: 'mycontroller')]
class MyController extends BaseController {
    // → reachable at www.myapplication.com/mycontroller.html
}
{% endhighlight %}

Run `vendor/bin/ud-routes generate` to pick up the attribute — see <a href="{{site.baseurl}}/docs/routing">Routing</a> for the full generation and dispatch flow.

---

## GET request

### check_authorization_get_request

Override this method to restrict which users can make a GET request to this controller. Return `true` to allow, `false` to deny. Returning `false` renders a shared "unauthorized" page (`http_response_code(403)`) instead of calling `getRequest()` — same mechanism for both GET and POST, see [Unauthorized access](#unauthorized-access) below.

Check by session group:

{% highlight php %}
public function check_authorization_get_request() {
    return isset($_SESSION['group'])
        && in_array($_SESSION['group'], ['readergroup', 'writergroup']);
}
{% endhighlight %}

Check ownership via the database:

{% highlight php %}
public function check_authorization_get_request() {
    $bookDao = new BookDao();
    $bookDao->setDBH($this->dbconnection->getDBH());
    $mybook = $bookDao->getOneByFields(['id' => $this->getParameters['id']]);
    return $mybook->author_id === $_SESSION['user_id'];
}
{% endhighlight %}

### Validation and filter rules

Declare `$get_validation_rules` and `$get_filter_rules` as public properties to validate and filter GET parameters before `getRequest()` is called. The framework runs these automatically and populates `$this->getParameters` with the cleaned values.

{% highlight php %}
public $get_validation_rules = ['authorid' => 'required|integer'];
public $get_filter_rules     = ['authorid' => 'trim'];
{% endhighlight %}

Multiple rules for the same field are separated by a pipe. Rules that take a parameter use a comma:

{% highlight php %}
public $get_validation_rules = [
    'id'   => 'required|integer',
    'slug' => 'required|alpha_numeric_dash|max_len,100',
];
public $get_filter_rules = [
    'id'   => 'trim',
    'slug' => 'trim|sanitize_string|lowercase',
];
{% endhighlight %}

See [Validation]({{site.baseurl}}/baseresources/validation) for the full list of available rules and filters.

### getRequest

Override this method to query the database and prepare data for the view. Every property you assign on `$this` becomes a variable available in the view file.

{% highlight php %}
public function getRequest() {
    $bookDao = new BookDao();
    $bookDao->setDBH($this->dbconnection->getDBH());
    $this->books  = $bookDao->getByFields(['author_id' => $this->getParameters['authorid']]);
    $this->author = 'William Shakespeare';
}
{% endhighlight %}

### View file

The view file renders the data prepared by `getRequest()`. It is loaded from the path defined in `$this->viewFile` with `Get.php` appended — e.g. `src/Chapters/Website/Views/MyControllerGet.php`.

{% highlight php %}
<h4><?= $author ?></h4>
<ul>
<?php foreach ($books as $book): ?>
    <li><?= htmlspecialchars($book->title) ?></li>
<?php endforeach; ?>
</ul>
{% endhighlight %}

### Handling validation errors

When GET parameters fail validation the framework calls `show_get_error_page()` instead of `getRequest()` and loads `MyControllerGetError.php` as the view. Override the method to prepare any data the error view needs. The validation error message is available in `$this->readableErrors`.

{% highlight php %}
public function show_get_error_page() {
    $this->errorMessage = $this->readableErrors;
}
{% endhighlight %}

{% highlight php %}
{{!-- src/Chapters/Website/Views/MyControllerGetError.php --}}
<div class="alert alert-danger"><?= htmlspecialchars($errorMessage) ?></div>
{% endhighlight %}

---

## POST request

### check_authorization_post_request

Same concept as the GET version, but for POST requests. Return `true` to allow, `false` to deny.

{% highlight php %}
public function check_authorization_post_request() {
    return isset($_SESSION['group'])
        && in_array($_SESSION['group'], ['readergroup', 'writergroup']);
}
{% endhighlight %}

### Validation and filter rules

Declare `$post_validation_rules` and `$post_filter_rules` to validate and filter POST parameters before `postRequest()` is called. Validated values are available in `$this->postParameters`.

{% highlight php %}
public $post_validation_rules = [
    'title'  => 'required|max_len,255',
    'email'  => 'required|valid_email',
    'price'  => 'required|float|min_numeric,0',
    'avatar' => 'required_file|extension,jpg;jpeg;png;gif',
];
public $post_filter_rules = [
    'title'  => 'trim|sanitize_string',
    'email'  => 'trim|sanitize_email|lowercase',
    'price'  => 'trim|sanitize_floats',
];
{% endhighlight %}

See [Validation]({{site.baseurl}}/baseresources/validation) for the full list of available rules and filters.

### postRequest

Override this method to process the submitted data — save to the database, send emails, and so on. Every property assigned on `$this` is available in the view.

{% highlight php %}
public function postRequest() {
    $bookDao = new BookDao();
    $bookDao->setDBH($this->dbconnection->getDBH());
    $bookDao->insert([
        'title'     => $this->postParameters['title'],
        'author_id' => $_SESSION['user_id'],
    ]);
    $this->redirectToPreviousPage();
}
{% endhighlight %}

### View file

Loaded from `$this->viewFile` with `Post.php` appended — e.g. `src/Chapters/Website/Views/MyControllerPost.php`.

{% highlight php %}
<p>Book saved successfully.</p>
{% endhighlight %}

### Handling validation errors

When POST parameters fail validation the framework calls `show_post_error_page()` and loads `MyControllerPostError.php`. The validation error message is available in `$this->readableErrors`.

{% highlight php %}
public function show_post_error_page() {
    $this->errorMessage = $this->readableErrors;
}
{% endhighlight %}

{% highlight php %}
{{!-- src/Chapters/Website/Views/MyControllerPostError.php --}}
<div class="alert alert-danger"><?= htmlspecialchars($errorMessage) ?></div>
{% endhighlight %}

---

## Unauthorized access

When `check_authorization_get_request()` or `check_authorization_post_request()` returns `false`, the controller renders a shared unauthorized page rather than a redirect — `http_response_code(403)` plus a static view, the same mechanism used for an unmatched route (see [Routing]({{site.baseurl}}/docs/routing)) but with its own template/view and a 403 instead of a 404.

| Method | Description |
|---|---|
| `show_unauthorized_page()` | Renders the unauthorized page. Called automatically on authorization failure; override only if you need custom behavior beyond the default. |
| `setUnauthorizedView($templateFile, $viewFile)` | Overrides the template/view used for the unauthorized page. Defaults to `'staticpage'` / `'errors/unauthorized'` — `'staticpage'` is a minimal built-in template that just requires the view file directly, so write `errors/unauthorized.php` as a normal, self-contained HTML page. |

{% highlight php %}
public function __construct() {
    parent::__construct();
    // ...
    // wrap the unauthorized page in the app's own chrome instead of the bare default
    $this->setUnauthorizedView('websitetemplate', 'errors/websiteunauthorized');
}
{% endhighlight %}

This is distinct from an invalid session (not logged in at all), which redirects to the application's login page before the controller is even reached.

---

## Redirect

After a POST request it is common to redirect rather than render a view. Four helper methods are available:

| Method | Description |
|---|---|
| `redirectToPreviousPage()` | Redirect to the last GET request the user made. |
| `redirectToSecondPreviousPage()` | Redirect to the GET request before that. |
| `redirectToPage($url)` | Redirect to a specific URL. If the constant `BASE_PATH` is defined it is prepended automatically. |
| `redirectToDefaultPage()` | Redirect to the application's default page defined in `DEFAULT_PAGE`. `BASE_PATH` is prepended if defined. |

{% highlight php %}
public function postRequest() {
    // ... save data ...
    $this->setSuccess('Record saved successfully.');
    $this->redirectToPreviousPage();
}
{% endhighlight %}

---

## Flash messages

You can pass a one-request message to the next page using these methods, which persist the value in the session until it is read:

| Method | Description |
|---|---|
| `setSuccess($msg)` | Green success message. |
| `setError($msg)` | Red error message. |
| `setInfo($msg)` | Blue informational message. |
| `setWarning($msg)` | Yellow warning message. |

The messages are automatically picked up by the messages block in the next controller's `makeAllPresets()` and made available in the view.
