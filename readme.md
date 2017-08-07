# RD Mailform

You need to take the sources from the repository file js/core.min.js

To initialize the form, you need to insert this JS code.

		var plugins = {
		    rdMailForm: $(".rd-mailform"),
		    regula: $("[data-constraints]"),
		    rdInputLabel: $(".form-label")
		  },
		  $document = $(document);

		$document.ready(function () {
			var isNoviBuilder = winodw.xMode;

			/**	
		   * attachFormValidator
		   * @description  attach form validation to elements
		   */
		  function attachFormValidator(elements) {
		      for (var i = 0; i < elements.length; i++) {
		        var o = $(elements[i]), v;
		        o.addClass("form-control-has-validation").after("<span class='form-validation'></span>");
		        v = o.parent().find(".form-validation");
		        if (v.is(":last-child")) {
		          o.addClass("form-control-last-child");
		        }
		      }

		      elements
		        .on('input change propertychange blur', function (e) {
		          var $this = $(this), results;

		          if (e.type !== "blur") {
		            if (!$this.parent().hasClass("has-error")) {
		              return;
		            }
		          }

		          if ($this.parents('.rd-mailform').hasClass('success')) {
		            return;
		          }

		          if ((results = $this.regula('validate')).length) {
		            for (i = 0; i < results.length; i++) {
		              $this.siblings(".form-validation").text(results[i].message).parent().addClass("has-error")
		            }
		          } else {
		            $this.siblings(".form-validation").text("").parent().removeClass("has-error")
		          }
		        })
		        .regula('bind');

		      var regularConstraintsMessages = [
		        {
		          type: regula.Constraint.Required,
		          newMessage: "The text field is required."
		        },
		        {
		          type: regula.Constraint.Email,
		          newMessage: "The email is not a valid email."
		        },
		        {
		          type: regula.Constraint.Numeric,
		          newMessage: "Only numbers are required"
		        },
		        {
		          type: regula.Constraint.Selected,
		          newMessage: "Please choose an option."
		        }
		      ];


		      for (var i = 0; i < regularConstraintsMessages.length; i++) {
		        var regularConstraint = regularConstraintsMessages[i];

		        regula.override({
		          constraintType: regularConstraint.type,
		          defaultMessage: regularConstraint.newMessage
		        });
		      }
		  }


		  /**
		   * isValidated
		   * @description  check if all elements pass validation
		   */
		  function isValidated(elements, captcha) {
		      var results, errors = 0;

		      if (elements.length) {
		        for (j = 0; j < elements.length; j++) {

		          var $input = $(elements[j]);
		          if ((results = $input.regula('validate')).length) {
		            for (k = 0; k < results.length; k++) {
		              errors++;
		              $input.siblings(".form-validation").text(results[k].message).parent().addClass("has-error");
		            }
		          } else {
		            $input.siblings(".form-validation").text("").parent().removeClass("has-error")
		          }
		        }

		        if (captcha) {
		          if (captcha.length) {
		            return validateReCaptcha(captcha) && errors === 0
		          }
		        }

		        return errors === 0;
		      }
		      return true;
		  }

		  /**
		   * RD Mailform
		   * @version      3.2.0
		   */
		  if (plugins.rdMailForm.length) {
		    var i, j, k,
		      msg = {
		        'MF000': 'Successfully sent!',
		        'MF001': 'Recipients are not set!',
		        'MF002': 'Form will not work locally!',
		        'MF003': 'Please, define email field in your form!',
		        'MF004': 'Please, define type of your form!',
		        'MF254': 'Something went wrong with PHPMailer!',
		        'MF255': 'Aw, snap! Something went wrong.'
		      };

		    for (i = 0; i < plugins.rdMailForm.length; i++) {
		      var $form = $(plugins.rdMailForm[i]),
		        formHasCaptcha = false;

		      $form.attr('novalidate', 'novalidate').ajaxForm({
		        data: {
		          "form-type": $form.attr("data-form-type") || "contact",
		          "counter": i
		        },
		        beforeSubmit: function (arr, $form, options) {
		          if (isNoviBuilder)
		            return;

		          var form = $(plugins.rdMailForm[this.extraData.counter]),
		            inputs = form.find("[data-constraints]"),
		            output = $("#" + form.attr("data-form-output")),
		            captcha = form.find('.recaptcha'),
		            captchaFlag = true;

		          output.removeClass("active error success");

		          if (isValidated(inputs, captcha)) {

		            // veify reCaptcha
		            if(captcha.length) {
		              var captchaToken = captcha.find('.g-recaptcha-response').val(),
		                captchaMsg = {
		                  'CPT001': 'Please, setup you "site key" and "secret key" of reCaptcha',
		                  'CPT002': 'Something wrong with google reCaptcha'
		                };

		              formHasCaptcha = true;

		              $.ajax({
		                method: "POST",
		                url: "bat/reCaptcha.php",
		                data: {'g-recaptcha-response': captchaToken},
		                async: false
		              })
		                .done(function (responceCode) {
		                  if (responceCode !== 'CPT000') {
		                    if (output.hasClass("snackbars")) {
		                      output.html('<p><span class="icon text-middle mdi mdi-check icon-xxs"></span><span>' + captchaMsg[responceCode] + '</span></p>')

		                      setTimeout(function () {
		                        output.removeClass("active");
		                      }, 3500);

		                      captchaFlag = false;
		                    } else {
		                      output.html(captchaMsg[responceCode]);
		                    }

		                    output.addClass("active");
		                  }
		                });
		            }

		            if(!captchaFlag) {
		              return false;
		            }

		            form.addClass('form-in-process');

		            if (output.hasClass("snackbars")) {
		              output.html('<p><span class="icon text-middle fa fa-circle-o-notch fa-spin icon-xxs"></span><span>Sending</span></p>');
		              output.addClass("active");
		            }
		          } else {
		            return false;
		          }
		        },
		        error: function (result) {
		          if (isNoviBuilder)
		            return;

		          var output = $("#" + $(plugins.rdMailForm[this.extraData.counter]).attr("data-form-output")),
		            form = $(plugins.rdMailForm[this.extraData.counter]);

		          output.text(msg[result]);
		          form.removeClass('form-in-process');

		          if(formHasCaptcha) {
		            grecaptcha.reset();
		          }
		        },
		        success: function (result) {
		          if (isNoviBuilder)
		            return;

		          var form = $(plugins.rdMailForm[this.extraData.counter]),
		            output = $("#" + form.attr("data-form-output")),
		            select = form.find('select');

		          form
		            .addClass('success')
		            .removeClass('form-in-process');

		          if(formHasCaptcha) {
		            grecaptcha.reset();
		          }

		          result = result.length === 5 ? result : 'MF255';
		          output.text(msg[result]);

		          if (result === "MF000") {
		            if (output.hasClass("snackbars")) {
		              output.html('<p><span class="icon text-middle mdi mdi-check icon-xxs"></span><span>' + msg[result] + '</span></p>');
		            } else {
		              output.addClass("active success");
		            }
		          } else {
		            if (output.hasClass("snackbars")) {
		              output.html(' <p class="snackbars-left"><span class="icon icon-xxs mdi mdi-alert-outline text-middle"></span><span>' + msg[result] + '</span></p>');
		            } else {
		              output.addClass("active error");
		            }
		          }

		          form.clearForm();

		          if (select.length){
		            select.select2("val", "");
		          }

		          form.find('input, textarea').trigger('blur');

		          setTimeout(function () {
		            output.removeClass("active error success");
		            form.removeClass('success');
		          }, 3500);
		        }
		      });
		    }
		  }


		  /**
		   * RD Input Label
		   * @description Enables RD Input Label Plugin
		   */

		  if (plugins.rdInputLabel.length) {
		    plugins.rdInputLabel.RDInputLabel();
		  }
	  });


And markup:

		<!-- RD Mailform-->
		<form class="rd-mailform" data-form-output="form-output-global" data-form-type="contact" method="post" action="bat/rd-mailform.php">
		  <div class="range range-20">
		    <div class="cell-sm-6">
		      <div class="form-wrap">
		        <input class="form-input" id="contact-name" type="text" name="name" data-constraints="@Required">
		        <label class="form-label" for="contact-name">Your Name</label>
		      </div>
		    </div>
		    <div class="cell-sm-6">
		      <div class="form-wrap">
		        <input class="form-input" id="contact-phone" type="text" name="phone" data-constraints="@Numeric">
		        <label class="form-label" for="contact-phone">Phone</label>
		      </div>
		    </div>
		    <div class="cell-xs-12">
		      <div class="form-wrap">
		        <label class="form-label" for="contact-message">Your Message</label>
		        <textarea class="form-input" id="contact-message" name="message" data-constraints="@Required"></textarea>
		      </div>
		    </div>
		    <div class="cell-sm-6">
		      <div class="form-wrap">
		        <input class="form-input" id="contact-email" type="email" name="email" data-constraints="@Email @Required">
		        <label class="form-label" for="contact-email">E-mail</label>
		      </div>
		    </div>
		    <div class="cell-sm-6">
		      <button class="button button-block button-primary" type="submit">Send Message</button>
		    </div>
		  </div>
		</form>

		<!-- Element for mailform error output -->
		<div class="form-output-global"></div>

Where are the following settings, on the form element and input:
* data-form-output - Id of the element where the result of sending will be displayed.
* data-form-type - Form type, can be: contact, subscribe, order
* data-constraints - On the form input fields and used for its validation. More setting here - https://github.com/vivin/regula/wiki

## Configuring the form's information messages

RD Mailform supports custom messages that inform about successful/unsuccessful sending of the form.

To change some default information messages, change the value of the msg field of the target validation type in script initialization file (scripts.js) in RD Mailform module:

        msg = {
          'MF000': 'Successfully sent!',
          'MF001': 'Recipients are not set!',
          'MF002': 'Form will not work locally!',
          'MF003': 'Please, define email field in your form!',
          'MF004': 'Please, define type of your form!',
          'MF254': 'Something went wrong with PHPMailer!',
          'MF255': 'Aw, snap! Something went wrong.'
        };
    

In order to set up the email address to send the e-mails (sent with the help of RD Mailform @version 3.2.0), please replace in bat/rd-mailform.config.json file, value of recipientEmail field to recipient email.

![](http://puu.sh/x3V0M/505e567ba6.png)


## SMTP Settings

![](http://puu.sh/x3V1G/1ce22c81b6.png)

To configure sending e-mail using SMTP, you need to open the bat/rd-mailform.config.json file and set the value of the useSmtp field to true. Then, enter the settings of your SMTP into next object field.
