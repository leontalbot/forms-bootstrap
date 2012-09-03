# Forms-Bootstrap #
Forms-Bootstrap is a utility for creating web forms in Clojure, styled using 
[Twitter's Bootstrap CSS](http://twitter.github.com/bootstrap/). It is built to be used with the 
[Noir](https://github.com/noir-clojure/noir) web framework, 
[Enlive](https://github.com/cgrand/enlive) HTML templating, and validation using 
[Sandbar](https://github.com/brentonashworth/sandbar).
You can use forms-bootstrap to quickly make nicely styled forms for your web app. It is easy to validate 
your forms and display well formatted error messages next to the appropriate fields. You can also pre-populate 
the form with default data from a source of your choice.

Forms-Bootstrap uses Twitter Bootstrap Version: 2.1

## Usage ##
Forms-Bootstrap is hosted on Clojars. Add

	[forms-bootstrap "0.2.0-SNAPSHOT"]

to your project.clj.

## Supported Form Controls / Styles ##
Forms-Bootstrap currently supports all the form controls and form styles listed on 
[Twitter Bootstrap's Forms Page](http://twitter.github.com/bootstrap/base-css.html#forms), with the exception of prepended and appended inputs. Check out that page to get an idea of what styles are available. Here is a brief summary:

Forms can have class 'inline form' for forms that go left to right, or class 'horizontal form' (default) for forms that go down the page. Specify the form class with :class. You can also specify a legend for the form with :legend.

Supported form controls: 

* input (text or password)
* select (dropdown menus)
* text area
* checkboxes
* radio buttons
* file input

Sizes (specified with :size):

* Relative Sizing: input-mini, input-small, input-medium, input-large, input-xlarge, input-xxlarge
* Grid Sizing: span1, span2, span3, ... span12

We also support inline-fields in a horizontal form (via the "controls-row" CSS class.) For example, you could have three selects in a row to get a user's birthday (day, month, year.) You can also add 'inline' to the radio or checkbox class to make them inline.

Form actions: just specify :submit-label and :cancel-label for a horizontal form, it defaults to 'btn-primary' (blue button style). For an inline form specify :submit-label and :button-type (default button type is 'btn'-- the grey button).

Help text: 'help-inline' (text to the right of the form field) and 'help-block' (text below form field) on any form field. Add these to :type.

Form control states: Add :disabled true to disable a form field.

Validation states: Add 'error' to the :type of the form-field. If validation fails, this happens automatically and the error message is displayed with 'error' styling. 

## Examples ##
Once you have cloned the project, navigate to the project directory and:
`lein run`

This lets you view the examples in /test. Open your favorite browser and go to:
`http://localhost:8080/`

## Snippets ##
Forms Bootstrap defines the following snippets listed below. (See 
[Enlive](https://github.com/cgrand/enlive) Templating). Snippets are functions that return 
a collection of nodes. You can use them with Enlive templates to get a string of HTML 
for use in a Noir defpage. Each form control has a 'field' version and a 'lite' version (ie: input-field and input-lite). The 'lite' version only makes the actual form control, whereas the 'field' version wraps it in a 'controls' and a 'control group' div for use in horizontal forms.

* basic-form: This is the 'main' snippet that takes in the sequence of 'nodes' generated by the other snippets
that each correspond to a form field type.
* input-field and input-lite
* text-area-field and text-area-lite
* select-field and select-lite
* checkbox-or-radio and checkbox-or-radio-lite
* file-input and file-input-lite
* make-submit-button and button-lite

## Form-Helper Macro ##
The form-helper macro is really convenient for making forms very quickly. You can call the macro with a name, a
collection of form fields, a validator function, a submit button label, a url to POST to, and functions to 
execute on successful validation and on failed validation. Form-Helper will then create a function bound to the name
you provided the macro. This function can be called with an action (ie users/dpetrovics/edit), a cancel link 
(ie: /users), and a form parameters map (used to pre-populate the form after a page render on failed validation, 
or to provide default values). When called, this function will return the enlive node representation of your form. 
You can use this Enlive map in a deftemplate to put the form into your desired html page, ie: 
(content your-form-name-here). The form-helper macro also creates the POST handler using Noir's defpage. Its route
is the POST url that you provided to the macro and it takes the form params as an argument. The POST handler's body
validates the form params, and then executes either the on-success or on-failure function.

Each 'field' in the :fields portion of a form-helper macro call can contain properties such as: 

* name: This assigns a name to the form element
* type: Supported types: text, password, text-area, select, checkbox, radio, file-input. 
* size: Supported sizes: input-mini, input-small, input-medium, input-large, input-xlarge, input-xxlarge, 
as well as span1, span2, span3, etc
* label: What to display next to the form element
* inputs: A vector of [[value1 displayedtext1] [value2 displayedtext2] ...]. Used by select, checkbox and radio.
* rows: Defines the number of rows for a text area
* text-area-content: A string of default text in a text area

Note: There are quite a few more options that you can provide depending on the form field. You can checkout core.clj 
and browse the arguments that each snippet takes.

Here is an example:

      (form-helper example-form
         :validator some-validator-fn-here
         :post-url "/users/:username/edit"
         :submit-label "Edit"
         :fields [{:name "first-name"
                   :label "First Name"
                   :type "text"}
                  {:name "gender"
                   :label "Gender"
                   :type "radio"   //can also be 'radio inline'
                   :inputs [["male" "Male"]		//first thing is the value, second is the label
                            ["female" "Female"]]}
                  {:name "email"
                   :label "Email Address"
                   :placeholder "user@site.com"
                   :type "text"}
				  {:type "inline-fields"	//used to create several form controls in one row, in horizontal forms
                   :name "birthday"
                   :label "Birthday"
                   :columns [{:name "birthday-day"
                              :type "select"
                              :size "input-small" 
                              :inputs [["" "Day"] ["1" "1"] ["2" "2"] ["3" "3"]]}
                             {:name "birthday-month"
                              :type "select"
                              :size "input-small"
                              :inputs [["" "Month"] ["1" "1"] ["2" "2"] ["3" "3"]]}
                             {:name "birthday-year"
                              :type "select"
                              :size "input-small"
                              :inputs [["" "Year"] ["2012" "2012"] ["2011" "2011"] ["2010" "2010"]]}]}
                  {:name "username"
                   :label "Username"
                   :type "text"}
                  {:name "password"
                   :label "Password"
                   :type "password"}]
         :on-success (fn [{uname :username :as user-map}]
                        (user/edit! user-map)
                        (session/flash-put! :flash "User edited successfully.")
                        (response/redirect "/"))
         :on-failure (fn [form-data]
                        (session/flash-put! :flash "Please Fix Errors")
                        (render "/users/:username/edit" form-data)))
   
Then to use the generated 'example-form' function:

      (defpage "/users/:username/edit"    ;;this is the 'GET' defpage. The 'POST' one is created for you by form-helper
         {:keys[username] :as m}
         (your-enlive-deftemplate-here 
            (example-form m   ;;form params map (or default inputs)
                          (str "users/" username "/edit")    ;;the action for the form
                          "/users")))  ;;cancel button link

Your Enlive HTML template should make sure to link to Twitter Bootstrap CSS to make use of their styling.

## TO DO ##
* Should we stick with Sandbar, or move the error handling over to Noir?
* Deal with empty checkboxes, radios, or dropdowns. In this situations the name of the form field is not present in the form params map, so we need a new way of searching for errors (in create-errors-defaults-map).
* More testing.

## License ##

Copyright (C) 2012 David Petrovics

Distributed under the Eclipse Public License, the same as Clojure.
