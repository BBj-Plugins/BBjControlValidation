# BBjControlValidation Plugin

<p>
  <a href="http://www.basis.cloud/downloads">
    <img src="https://img.shields.io/badge/BBj-v22.10-blue" alt="BBj v22.10" />
  </a>
  <a href="http://www.basis.cloud/downloads">
    <img src="https://img.shields.io/badge/Client-DWC-blue" alt="Client DWC" />
  </a>    
  <a href="https://github.com/BBj-Plugins/BBjControlValidation/blob/master/README.md">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="BBjControlValidation is released under the MIT license." />
  </a>
  <a href="https://github.com/necolas/issue-guidelines/blob/master/CONTRIBUTING.md#pull-requests">
    <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs welcome!" />
  </a>
</p>

BBjControlValidation provides a set of Javascript validation expressions for use with BBjControls.

?> **Note:** For more information about form validations in BBj. please [start here](https://basishub.github.io/basis-next/#/dwc/form-validation)

## Features

* Easy to set up
* Easy to customize
* 30+ of common validators
* Ability to extend and add new validator types

## Installation

- Clone the [project](https://github.com/BBj-Plugins/BBjControlValidation) locally , then add `BBjControlValidation` to your BBj paths
- Or [Use the plugins manager](https://www.bbj-plugins.com/en/get-started)

## The gist


<div style="text-align: center;">
  <img style="border:thin solid var(--bbj-color-default); border-radius: var(--bbj-border-radius-m)" src="assets/validation-form.png" alt="A simple form with validation built using the the `ValidationBuilder`">
  <em>A simple registration form with validation built using the BBjControlValidation</em>
</div>
<br><br>


```BBj
use com.google.gson.JsonObject
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

rem Validation constraints
rem =======================
emailValidator! = (new ValidationBuilder()).notBlank().email()
passwordValidator! = (new ValidationBuilder()).notBlank().password().match(".confirm-password-editbox", "The password does not match")
confirmValidator! = (new ValidationBuilder()).notBlank().match(".password-editbox", "The password does not match")
checkedValidator! = (new ValidationBuilder()).isTrue("You must accept the terms and conditions")

rem styles
rem =======
outerStyle! = new JsonObject()
outerStyle!.addProperty("position", "absolute")
outerStyle!.addProperty("top", "50%")
outerStyle!.addProperty("left", "50%")
outerStyle!.addProperty("padding", "var(--bbj-space-m)")
outerStyle!.addProperty("transform", "translateX(-50%) translateY(-50%)")

panelStyle! = new JsonObject()
panelStyle!.addProperty("width", "300px")
panelStyle!.addProperty("height", "100%")
panelStyle!.addProperty("padding", "var(--bbj-space)")
panelStyle!.addProperty("display", "grid")
panelStyle!.addProperty("gap", "var(--bbj-space-m)")
panelStyle!.addProperty("grid-template-columns", "auto")
panelStyle!.addProperty("justify-items", "stretch")
panelStyle!.addProperty("align-items", "stretch")

rem UI elements
rem ============
sysgui = unt
open (sysgui)"X0"
sysgui! = bbjapi().getSysGui()
window! = sysgui!.addWindow(25,25,300,120,"Create New Account",$00190000$)
window!.setTitleBarVisible(BBjAPI.FALSE)
window!.setOuterStyle("@element", outerStyle!.toString())
window!.setPanelStyle("@element", panelStyle!.toString())
window!.setCallback(window!.ON_CLOSE, "handleClose")

title! = window!.addStaticText("<html><h1 style='text-align:center'>Create New Account</h1></html>")

email! = window!.addEditBox("")
email!.setAttribute("label", "Email Address")
email!.setAttribute("placeholder", "Your email address")
email!.setClientValidationFunction(str(emailValidator!))

password! = window!.addEditBox("")
password!.addStyle("password-editbox")
password!.setAttribute("type", "password")
password!.setAttribute("label", "Password")
password!.setAttribute("placeholder", "password")
password!.setClientValidationFunction(str(passwordValidator!))

confirm! = window!.addEditBox("")
confirm!.addStyle("confirm-password-editbox")
confirm!.setAttribute("type", "password")
confirm!.setAttribute("label", "Confirm Password")
confirm!.setAttribute("placeholder", "Confirm password")
confirm!.setClientValidationFunction(str(confirmValidator!))

accept! = window!.addCheckBox("Agree to the terms and conditions as set out by the user agreement.")
accept!.addStyle("bbj-whitespace-wrap")
accept!.setClientValidationFunction(str(checkedValidator!))

submit! = window!.addButton("Submit")
submit!.setStyle("margin-top", "var(--bbj-space-m)")
submit!.setAttribute("expanse", "m")
submit!.setAttribute("theme", "primary")
submit!.setCallback(submit!.ON_FORM_VALIDATION, "handleSubmit")

email!.focus()

process_events

handleSubmit:
  event! = sysgui!.getLastEvent()
  event!.accept(1)
  a! = msgbox("Your account has been created successfully.", 0, "Account created", mode="theme=primary,blurred")
return

handleClose:
release
```

## Violations Accumulation

By default the `ValidationBuilder` will accumulate all errors and display them at once. It is possible to disable this feature 
by passing `false` to the constructor then only one violation message will be displayed at once. 

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = new ValidationBuilder(BBjAPI.FALSE)
```

## Managing Validators

The `ValidationBuilder` provide two methods to add/remove validators. 

1. `add(::BBjControlValidation/Validator.bbj::Validator validator!)`
2. `remove(::BBjControlValidation/Validator.bbj::Validator validator!)`

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder
use ::BBjControlValidation/Validators/NotBlank.bbj::NotBlank
use ::BBjControlValidation/Validators/MinLength.bbj::MinLength
use ::BBjControlValidation/Validators/MaxLength.bbj::MaxLength
use ::BBjControlValidation/Validators/Regex.bbj::Regex

validation! = new ValidationBuilder()
validation!.add(new NotBlank("Please provide a username"))
validation!.add(new MinLength(2))
validation!.add(new MaxLength(20))
validation!.add(new Regex("^[a-zA-Z0-9_]+$"))
...
control!.setClientValidationFunction(validation!.build())
```

### Custom Validators

In order to add new validators , you need a class which extends the `::BBjControlValidation/Validator.bbj::Validator`

For instance , here is a simple `Gmail` validator

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder
use ::BBjControlValidation/Validator.bbj::Validator

class public GmailEmailValidator extends Validator
  rem /**
  rem  * @param message$ - The invalid message
  rem  */
  method public GmailEmailValidator(BBjString message$)
    #super!(message$, "")
    #Validator$ = "" +
:    "if (!/^.+\@\gmail.com+$/.test(value)) {" +
:    " addViolation('" + message$ + "');" +
:    "}"
  methodend

classend

validation! = new ValidationBuilder()
validation!.add(new GmailEmailValidator("Please provide a valid Gmail address"))
...
control!.setClientValidationFunction(validation!.build())
```

# Available Validators

## Blank

Validates that the control's value is blank - meaning equals to a blank string or an empty array

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).blank("Optional Message")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |


## NotBlank

Validates that the control's value is not blank - meaning not equals to a blank string or an empty array

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).notBlank("Optional Message")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## IsTrue

Validates that the control's value is true.

?> **Note:** When working with [BBjCheckBox](https://documentation.basis.cloud/BASISHelp/WebHelp/bbjobjects/Window/bbjcheckbox/BBjCheckBox.htm?Highlight=BBjCheckBox) and [BBjRadioButton](https://documentation.basis.cloud/BASISHelp/WebHelp/bbjobjects/Window/bbjradiobutton/BBjRadioButton.htm?Highlight=BBjRadio) you can use `ValidationBuilder.checked()` method as an alias to `ValidationBuilder.isTrue()`. The different is that the alias has a different default message.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).isTrue()
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## IsFalse

Validates that the control's value is false.

?> **Note:** When working with [BBjCheckBox](https://documentation.basis.cloud/BASISHelp/WebHelp/bbjobjects/Window/bbjcheckbox/BBjCheckBox.htm?Highlight=BBjCheckBox) and [BBjRadioButton](https://documentation.basis.cloud/BASISHelp/WebHelp/bbjobjects/Window/bbjradiobutton/BBjRadioButton.htm?Highlight=BBjRadio) you can use `ValidationBuilder.notChecked()` method as an alias to `ValidationBuilder.isFalse()`. The different is that the alias has a different default message.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).isFalse()
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## Length

Validates that the control's value length equal to a given length.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).length(10, "The value should be {{length}} characters long.")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `length`     |    The length of the string   |

## MinLength

Validates that the control value's length is at least `x` characters long

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).minLength(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `min`     |   The minimum length of the string |

## MaxLength

Validates that the control value's length is equal or greater than the specified length.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).maxLength(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `max`     |    The maximum length of the string   |

## Count

Validates that the control's value element count is equal to the specified length. For controls which their value is an array like [BBjListBox](https://documentation.basis.cloud/BASISHelp/WebHelp/bbjobjects/Window/bbjlistbox/BBjListBox.htm?Highlight=BBjListBox)

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).count(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `count`     |    The length to compare the control's value element count to    |

## MinCount

Validates that the control's value element count is at least the specified minimum. For controls which their value is an array like [BBjListBox](https://documentation.basis.cloud/BASISHelp/WebHelp/bbjobjects/Window/bbjlistbox/BBjListBox.htm?Highlight=BBjListBox)

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).minCount(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `min`     |   The minimum length to compare the control's value element count to     |

## MaxCount

Validates that the control's value element count is at least the specified minimum. For controls which their value is an array like [BBjListBox](https://documentation.basis.cloud/BASISHelp/WebHelp/bbjobjects/Window/bbjlistbox/BBjListBox.htm?Highlight=BBjListBox)

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).maxCount(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `max`     |   The maximum length to compare the control's value element count to     |

## EqualTo

Validates that the control's value is equal to the given value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).equalTo("Foobar")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## NotEqualTo

Validates that the control's value is not equal to the given value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).notEqualTo("Foobar")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## GreaterThan

Validates that the value is greater than the specified value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).greaterThan(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## GreaterThanOrEqual

Validates that the value is greater than or equals to the specified value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).greaterThanOrEqual(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## LessThan

Validates that the value is less than the specified value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).lessThan(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## LessThanOrEqual

Validates that the value is less than or equals the specified value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).lessThanOrEqual(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## DivisibleBy

Validates that the value is divisible by the given number.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).divisibleBy(5)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |


## Regex

Validates that the control's value matches a regular expression.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).regex("^\d+$", "ig", "The value does not match {{regex}}")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |

## Email

Validates that the control's value is a valid email address

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).email("Not valid email")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |


## Password

Validates that the control's value is a valid password (UpperCase, LowerCase, Number/SpecialChar and minimum `x` Chars)

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).password(10)
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |
|   `length`     |    The password's min length    |

## DateTime

Validates that the control's value is a valid datetime, meaning a string that follows a valid input type="datetime" format.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

dateTimeValidator! = (new ValidationBuilder()).datetime()
control!.setClientValidationFunction(str(dateTimeValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |

## Date

Validates that the control's value is a valid date, meaning a string that follows a valid `YYYY-MM-DD` format.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

dateValidator! = (new ValidationBuilder()).date()
control!.setClientValidationFunction(str(dateValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |

## PastDate

Validates that the controls's value is a date in the past. The time portion of the date will be ignored.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validator! = (new ValidationBuilder()).pastDate()
control!.setClientValidationFunction(str(validator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## FutureDate

Validates that the controls's value is a date in the future. The time portion of the date will be ignored.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validator! = (new ValidationBuilder()).futureDate()
control!.setClientValidationFunction(str(validator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |


## Time

Validates that the control's value is a valid time, meaning a string that follows a valid HH:MM:SS format.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

timeValidator! = (new ValidationBuilder()).time()
control!.setClientValidationFunction(str(timeValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |

## Match 

Validates that a control's value matches the value of a second control which can be found with the given selector

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).match(".password-editbox", "The password does not match")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## Choice

Validates that that the control's value is one of a given set of valid choices

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

choices! =  new BBjVector()
choices!.addItem("ListButton Item 2")
choices!.addItem("ListButton Item 4")

choiceValidator! = (new ValidationBuilder()).choice(choices!)
control!.setClientValidationFunction(str(choiceValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## Json

Validates that the control's value is a valid json string

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).json()
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## IPv4

Validates that a value is a valid IPv4 address

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).ipv4()
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |

## IPv6

Validates that a value is a valid IPv6 address

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).ipv6()
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |

## AlphaNumeric

Validates that the control's value contains only alphabetical and numerical characters.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).alphaNumeric()
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |

## Host

Validates that the control's value is valid URL containing the domain name.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).host("example.com, foo.bar")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## Protocol

Validates that the control's value is valid URL with the given protocol.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).protocol("sftp: , ssh:")
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |


## UUID

Validates that the control's value is a valid UUID

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validation! = (new ValidationBuilder()).uuid()
control!.setClientValidationFunction(validation!.build())
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |
