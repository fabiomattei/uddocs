---
layout: page
title: How to create an Ajax form
orderfield: 8
---

An ajax form is a form that does not have a submit button.
An Ajax form has one or more <a href="{{site.baseurl}}/baseresources/ajaxbuttons">Ajax buttons</a> and, if clicked, does not reload the whole page,
An Ajax form only relad a section of a page. 

### The Json version

In the following example we can see an Ajax form that allows the user to change his address and his phone number. 
This form is not that different from a <a href="{{site.baseurl}}/resources/form">form</a> resource.

There are tree main differences:
* the **formid** section
* the **bottombuttons** section containing <a href="{{site.baseurl}}/baseresources/ajaxbuttons">Ajax buttons</a>
* the <a href="{{site.baseurl}}/baseresources/ajaxreponses">Ajax response</a> section

{% highlight json %}
{
  "name": "profile-consumer-edit-form",
  "metadata": { "type":"form-extended", "version": "1" },
  "allowedgroups": [ "consumergroup", "resellergroup", "admingroup" ],
  "get":{
    "query": {
      "sql": "SELECT * FROM users WHERE us_uuid = :id;",
      "parameters":[
        { "type":"string", "placeholder": ":id", "sessionparameter": "user_id" }
      ]
    },
    "form": {
      "formid": "profile-consumer-edit-form-id",
      "title": "Modyfy your profile",
      "fields": [
        { "type":"textfield",  "name":"name", "label":"Name", "sqlfield": "usr_name","size":"120", "width":"4", "row":"1", "readonly":"readonly" },
        { "type":"textfield",  "name":"email", "label":"Email", "sqlfield": "usr_email", "size":"120", "width":"4", "row":"1", "readonly":"readonly" },
        { "type":"textfield", "name":"address", "label":"Address", "sqlfield": "usr_address", "size":"120", "width":"4", "row":"2" },
        { "type":"textfield",  "name":"phone", "label":"Phone", "sqlfield": "usr_phone", "size":"120", "width":"4", "row":"2" }
      ],
      "bottombuttons": [
        {
          "type":"ajaxbutton",
          "cssclass":"udbuttonajaxrequest btn btn-success btn-sm",
          "icon":"bi bi-file-earmark-plus-fill",
          "method": "POST",
          "dataudformid":"profile-consumer-edit-form-id",
          "dataudurl":{
            "controller": "partial",
            "resource":"profile-consumer-edit-form"
          }
        },
        {
          "type": "ajaxbutton",
          "icon": "bi bi-skip-backward-circle-fill",
          "cssclass": "udbuttonoverwrite btn btn-warning btn-sm",
          "method": "GET",
          "dataudiddestination": "#panel3",
          "dataudurl":{
            "controller": "partial",
            "resource":"profile-consumer-grid"
          }
        }
      ]
    }
  },
  "post" : {
    "request": {
      "postparameters": [
        { "type":"string", "validation":"max_len,250", "name":"phone" },
        { "type":"string", "validation":"max_len,250", "name":"address" }
      ]
    },
    "transactions": [
      {
        "sql":"UPDATE dbd_users SET us_phone = :phone, us_address = :address, us_updated = NOW() WHERE us_uuid = :id;",
        "parameters":[
          { "type":"string", "placeholder": ":id", "sessionparameter": "user_id" },
          { "type":"string", "placeholder": ":phone", "postparameter": "phone" },
          { "type":"string", "placeholder": ":address", "postparameter": "address" }
        ]
      }
    ],
    "ajaxreponses": [
      {
        "type": "overwriteurl",
        "destination": "#panel2",
        "method": "GET",
        "url": {
          "controller": "partial",
          "resource": "empty"
        }
      },
      {
        "type": "overwriteurl",
        "destination": "#panel1",
        "method": "GET",
        "url": {
          "controller": "partial",
          "resource": "profile-consumer-info"
        }
      }
    ]
  }
}
{% endhighlight %}

### The PHP version

#### Controller
{% highlight php %}
<?php

namespace DBYD\Chapters\Common\Controllers;

use DBYD\Chapters\Common\Controllers\BaseUserLoggedInController;
use DBYD\Chapters\CRM\Daos\UserDaoExtended;

class UserChangesPasswordController extends BaseUserLoggedInController
{

    const CONTROLLER_NAME = 'profile-reset-password';
    public string $footViewFile;
    public string $headViewFile;
    public $user;
    public $utenteExtra;
    public $errorMessage;

    /**
     * @param string[] $get_validation_rules
     */
    public function __construct()
    {
        parent::__construct();
        $this->classCompleteName = __CLASS__;
        $this->className = 'UserChangesPasswordController';
        $this->chapter = 'Common';
        $this->templateFile = 'apptemplate';
        $this->viewFile = 'src/Chapters/Common/Views/UserChangesPasswordController';
        $this->controllerPointer = $this;
        $this->appTitle = 'MCRM :: Impostazioni';
    }

    public function check_authorization_get_request()
    {
        return (isset($_SESSION['group']) and (in_array($_SESSION['group'], ['resellergroup', 'consumergroup'])));
    }

    public static function generateGetLink(int $taId = 0): string
    {
        return \DBYD\Chapters\Common\Controllers\UserChangesPasswordController::CONTROLLER_NAME . '.html';
    }

    public function getRequest()
    {
        /* parent::getRequest(); */
        $userDaoExtended = new UserDaoExtended;
        $userDaoExtended->setDBH($this->dbconnection->getDBH());
        $userDaoExtended->setLogger($this->logger);

        $this->utenteExtra = $userDaoExtended->getById($_SESSION['user_id']);

        $this->errorMessage = $_SESSION['msgerror'] ?? '';
    }

    public /* array */ $post_validation_rules = [
        'password' => 'between_len,1;255',
        'newpassword' => 'between_len,1;255',
        'repeatpassword' => 'between_len,1;255',
    ];

    public /* array */ $post_filter_rules = [
        'password' => 'trim',
        'newpassword' => 'trim',
        'repeatpassword' => 'trim',
    ];

    public function postRequest()
    {
        $this->templateFile = 'empty';
        $userDaoExtended = new UserDaoExtended;
        $userDaoExtended->setDBH($this->dbconnection->getDBH());
        $userDaoExtended->setLogger($this->logger);

        $user = $userDaoExtended->getById($_SESSION['user_id']);
        if ($userDaoExtended->checkUserNameAndPassword($user->usr_username, $this->postParameters['password'])) {
            if ($this->postParameters['newpassword'] == $this->postParameters['repeatpassword']) {
                $userDaoExtended->updatePassword($_SESSION['user_id'], $this->postParameters['newpassword']);
                $this->setSuccess("Password cambiata");
            } else {
                $this->setError("Le due password non corrispondono");
            }
        } else {
            $this->setError("Password fornita errata");
        }

        // Ajax response
        echo '[
        {"type":"overwriteurl","destination":"#panel2","position":"beforeend","url":"empty.html?udpartial=true&","method":"GET"},
        {"type":"overwriteurl","destination":"#panel1","position":"beforeend","url":"profile-'.substr($user->usr_defaultgroup, 0, -5).'-info.html?udpartial=true&","method":"GET"}
        ]';
    }
}

{% endhighlight %}

#### View

{% highlight php %}
<?php

use DBYD\Chapters\Common\Controllers\UserChangesPasswordController;

?>
<div class="card">
    <h5 class="card-header">Modifica password</h5>
    <div class="card-body">
        <form id="profile-reset-password" action="<?= UserChangesPasswordController::generateGetLink() ?>" method="POST">
            <div class="form-group">
                <label for="exampleInputPassword1">Password attuale</label>
                <input type="password" name="password" class="form-control" id="exampleInputPassword1"
                    placeholder="Password">
            </div>

            <div class="form-group">
                <label for="exampleInputPassword2">Nuova password</label>
                <input type="password" name="newpassword" class="form-control" id="exampleInputPassword2"
                    placeholder="Password">
            </div>

            <div class="form-group">
                <label for="exampleInputPassword3">Ripeti nuova password</label>
                <input type="password" name="repeatpassword" class="form-control" id="exampleInputPassword3"
                    placeholder="Password">
            </div>
        </form>
        <br />
        <button data-udformid="profile-reset-password" data-udurl="<?= UserChangesPasswordController::generateGetLink() ?>?udpartial=true" class="udbuttonajaxrequest btn btn-success btn-sm" data-udmethod="POST"><i class="bi bi-file-earmark-plus-fill" role="img"></i></button>
        <button data-udiddestination="#panel2" data-udurl="empty.html?udpartial=true" class="udbuttonoverwrite btn btn-warning btn-sm" data-udmethod="GET"><i class="bi bi-skip-backward-circle-fill" role="img"></i></button>
    </div>
</div>
{% endhighlight %}

