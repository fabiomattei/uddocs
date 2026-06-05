---
layout: page
name: Validation
---

# Validation

UglyDuckling validates and filters incoming request parameters automatically before your controller runs. You declare rules as public properties on your controller; the framework applies them for every GET and POST request.

The engine is the built-in `Validation` class (`Fabiom\UglyDuckling\Framework\Validation\Validation`), which replaced the GUMP library. It fixes a PHP 8.1+ issue where apostrophes and quotes were incorrectly encoded as HTML entities in stored data.

---

## Declaring rules in a controller

```php
class MyController extends Controller {

    public $get_validation_rules  = ['id' => 'required|integer'];
    public $get_filter_rules      = ['id' => 'trim'];

    public $post_validation_rules = [
        'name'  => 'required|max_len,255',
        'email' => 'required|valid_email',
    ];
    public $post_filter_rules     = [
        'name'  => 'trim|sanitize_string',
        'email' => 'trim|sanitize_email|lowercase',
    ];

    public function postRequest() {
        $name  = $this->postParameters['name'];
        $email = $this->postParameters['email'];
    }
}
```

If validation fails the framework calls `show_get_error_page()` or `show_post_error_page()` instead of `getRequest()` / `postRequest()`. The error message is available in `$this->readableErrors`.

---

## Declaring rules in a JSON resource

```json
"request": {
  "parameters": [
    { "name": "id",    "validation": "required|integer" },
    { "name": "email", "validation": "required|valid_email" }
  ]
}
```

---

## Rule syntax

Multiple rules for the same field are separated by a **pipe** character. Rules are evaluated left to right and stop at the first failure for each field.

```
"required|min_len,3|max_len,100"
```

Rules that take a parameter use a **comma**: `max_len,100`. Rules that take two parameters use a **semicolon** to separate them: `between_len,3;100`.

---

## Validation rules

### Presence

| Rule | Description |
|---|---|
| `required` | Field must be present and non-empty. |

### String length

| Rule | Description |
|---|---|
| `min_len,N` | Value must be at least N characters long. |
| `max_len,N` | Value must be at most N characters long. |
| `between_len,MIN;MAX` | Value length must be between MIN and MAX characters. |

### Character set

| Rule | Description |
|---|---|
| `alpha_numeric` | Only letters and digits allowed. |
| `alpha_numeric_dash` | Only letters, digits, hyphens and underscores allowed. |

### Numbers

| Rule | Description |
|---|---|
| `numeric` | Value must be numeric (integer or float). |
| `integer` | Value must be a whole number (no decimals). |
| `float` | Value must be a valid float number. |
| `integer_between,MIN;MAX` | Value must be an integer within \[MIN, MAX\] inclusive. |
| `min_numeric,N` | Value must be numeric and greater than or equal to N. |
| `max_numeric,N` | Value must be numeric and lower than or equal to N. |

### Format

| Rule | Description |
|---|---|
| `valid_email` | Value must be a syntactically valid email address. |
| `valid_url` | Value must be a syntactically valid URL. |
| `valid_date` | Value must be a valid date in `YYYY-MM-DD` format. |
| `past_date` | Value must be a valid `YYYY-MM-DD` date strictly before today. |
| `future_date` | Value must be a valid `YYYY-MM-DD` date strictly after today. |
| `strong_password` | Value must contain at least one uppercase letter, one lowercase letter, one digit and one special character. |

### File upload

| Rule | Description |
|---|---|
| `required_file` | A file must have been uploaded without errors. |
| `extension,ext1;ext2` | The uploaded file's extension must be in the semicolon-separated list (case-insensitive). |

For file uploads the framework merges `$_FILES` into the POST parameters before validation. Use `required_file` to make an upload mandatory; omit it to make the upload optional (the `extension` rule still validates when a file is present).

```php
// Required upload
public $post_validation_rules = ['avatar' => 'required_file|extension,jpg;jpeg;png;gif'];

// Optional upload — validated only when a file is provided
public $post_validation_rules = ['avatar' => 'extension,jpg;jpeg;png;gif'];
```

---

## Filter rules

Filters transform values **before** validation runs, so validators always see the cleaned value. Multiple filters per field are separated by a pipe character and applied left to right.

### Whitespace and encoding

| Filter | Description |
|---|---|
| `trim` | Remove leading and trailing whitespace. |
| `urlencode` | Percent-encode the string for safe use in a URL. |
| `htmlencode` | Convert HTML special characters to their HTML entities. |
| `lowercase` | Convert the value to lower case. |
| `uppercase` | Convert the value to upper case. |
| `slug` | Convert to a URL-friendly slug — e.g. `"Caffè Latte!"` → `"caffe-latte"`. Transliterates accented characters. |

### Sanitization

| Filter | Description |
|---|---|
| `sanitize_string` | Strip null bytes and HTML tags. Does **not** encode quotes. |
| `sanitize_email` | Remove characters that are not valid in an email address. |
| `sanitize_numbers` | Remove all characters that are not digits. |
| `sanitize_floats` | Remove all characters that are not digits, dot, plus or minus. |
| `rmpunctuation` | Remove all punctuation characters (Unicode-aware). |
| `basic_tags` | Strip all HTML tags except a safe subset: `<b>` `<i>` `<u>` `<p>` `<br>` `<strong>` `<em>` `<a>` `<ul>` `<ol>` `<li>` `<span>`. |

### Type conversion

| Filter | Description |
|---|---|
| `boolean` | Converts `1`, `'1'`, `'true'`, `'yes'`, `'on'` to `true`; everything else to `false`. Useful for checkboxes. |

---

## Common combinations

```php
// A required name field
'name' => 'required|min_len,2|max_len,100'
// filter:
'name' => 'trim|sanitize_string'

// A required email field
'email' => 'required|valid_email'
// filter:
'email' => 'trim|sanitize_email|lowercase'

// A password field with strength requirement
'password' => 'required|min_len,8|strong_password'

// An optional integer ID from a GET parameter
'id' => 'integer'
// filter:
'id' => 'trim'

// A positive price value
'price' => 'required|float|min_numeric,0.01|max_numeric,9999.99'

// An age that must be a past date
'birth_date' => 'required|past_date'

// A quantity between 1 and 100
'quantity' => 'required|integer_between,1;100'

// A URL-based slug generated automatically
// filter only — no validation rule needed:
'slug' => 'trim|slug'
```

---

## Multi-language error messages

Pass a language code to the `Validation` constructor. The framework defaults to `'en'`; `'it'` is also bundled.

```php
$this->gump = new Validation('it');
```

To add a new language create `src/Framework/Validation/lang/fr.php` returning an array with the same keys as `en.php` and translated message templates. Available placeholders are `{field}`, `{min}`, and `{max}`.

```php
// lang/fr.php
return [
    'required'  => 'Le champ {field} est obligatoire.',
    'max_len'   => 'Le champ {field} ne peut pas dépasser {max} caractères.',
    // ...
];
```
