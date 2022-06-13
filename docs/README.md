# BBjControlValidation Plugin

<p>
  <a href="http://www.basis.com/downloads">
    <img src="https://img.shields.io/badge/BBj-v22.10-blue" alt="BBj v22.10" />
  </a>
  <a href="https://github.com/BBj-Plugins/BBjControlValidation/blob/master/README.md">
    <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="BBjControlValidation is released under the MIT license." />
  </a>
  <a href="https://github.com/necolas/issue-guidelines/blob/master/CONTRIBUTING.md#pull-requests">
    <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg" alt="PRs welcome!" />
  </a>
</p>

BBjControlValidation provides a set of validation expressions for use in BBjControls.

## Installation

- Clone the [project](https://github.com/BBj-Plugins/BBjControlValidation) locally , then add `BBjControlValidation` to your BBj paths
- Or [Use the plugins manager](https://www.bbj-plugins.com/en/get-started)

## The gist

<div style="text-align: center;">
  <img style="border:thin solid var(--bbj-color-default); border-radius: var(--bbj-border-radius-m)" src="assets/validation-form.png" alt="BBjControlValidation gist">
</div>
<br><br>

```BBj
use com.google.gson.JsonObject
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

rem validation constraints
emailValidator! = (new ValidationBuilder()).notBlank().email()
passwordValidator! = (new ValidationBuilder()).notBlank().password().match(".confirm-password-editbox", "The password does not match")
confirmValidator! = (new ValidationBuilder()).notBlank().match(".password-editbox", "The password does not match")
checkedValidator! = (new ValidationBuilder()).isTrue("You must accept the terms and conditions")

rem styles
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
sysgui = unt
open (sysgui)"X0"
sysgui! = bbjapi().getSysGui()
window! = sysgui!.addWindow(25,25,300,120,"Create Account",$00190000$)
window!.setTitleBarVisible(BBjAPI.FALSE)
window!.setOuterStyle("@element", outerStyle!.toString())
window!.setPanelStyle("@element", panelStyle!.toString())
window!.setCallback(window!.ON_CLOSE, "handleClose")

title! = window!.addStaticText("<html><h1 style='text-align:center'>Create New Account</h1></html>")

control! = window!.addEditBox("")
control!.setAttribute("label", "Email Address")
control!.setAttribute("placeholder", "Your email address")
control!.setClientValidationFunction(str(emailValidator!))

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

control!.focus()

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
by passing `false` to the constructor.

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

validator! = new ValidationBuilder(BBjAPI.FALSE)
```

# Validators

## Blank

 Validates that control's value is not blank - meaning not equal to a blank string, false or null

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

blankValidator! = (new ValidationBuilder()).blank("optional-message")
control!.setClientValidationFunction(str(blankValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |


## NotBlank

Validates that the control's value is not blank - meaning not equal to a blank string, a false or null

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

notBlankValidator! = (new ValidationBuilder()).notBlank("optional-message")
control!.setClientValidationFunction(str(notBlankValidator!))
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

lengthValidator! = (new ValidationBuilder()).length(10, "The value should be {{length}} characters long.")
control!.setClientValidationFunction(str(lengthValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `length`     |    the length of the string   |

## MaxLength

Validates that the control's value length is less than or equal to a given length.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

lengthValidator! = (new ValidationBuilder()).maxLength(10)
control!.setClientValidationFunction(str(lengthValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `length`     |    the maximum length of the string   |

## MinLength

Validates that the control's value length is at least x characters

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

lengthValidator! = (new ValidationBuilder()).minLength(10)
control!.setClientValidationFunction(str(lengthValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `length`     |   the minimum length of the string |

## EqualTo

Validates that the control's value is equal to the given value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

eqValidator! = (new ValidationBuilder()).equalTo("BBj")
control!.setClientValidationFunction(str(eqValidator!))
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

notEqValidator! = (new ValidationBuilder()).notEqualTo("BBj")
control!.setClientValidationFunction(str(notEqValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## GreaterThan

Validates that the control's value is greater than the specified value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

gtValidator! = (new ValidationBuilder()).greaterThan(10)
control!.setClientValidationFunction(str(gtValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## GreaterThanOrEqual

Validates that the control's value is greater than the specified value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

gtOrEqValidator! = (new ValidationBuilder()).greaterThanOrEqual(10)
control!.setClientValidationFunction(str(gtOrEqValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## LessThan

Validates that the control's value is less than the specified value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

lsValidator! = (new ValidationBuilder()).lessThan(10)
control!.setClientValidationFunction(str(lsValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `comparedValue`     |    The value to compare with  |

## LessThanOrEqual

Validates that the control's value is less than or equals the specified value

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

lsOrEqValidator! = (new ValidationBuilder()).lessThanOrEqual(10)
control!.setClientValidationFunction(str(lsOrEqValidator!))
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

regexValidator! = (new ValidationBuilder()).regex("^\d+$", "ig", "The value does not match {{regex}}")
control!.setClientValidationFunction(str(regexValidator!))
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

emailValidator! = (new ValidationBuilder()).email("{{value}} is not a valid email address")
control!.setClientValidationFunction(str(emailValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |
|   `regex`     |    The regular expression    |
|   `flag`     |    The regular expression flags    |


## Password

Validates that the control's value is a valid password (UpperCase, LowerCase, Number/SpecialChar and min x Chars

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

passwordValidator! = (new ValidationBuilder()).password(8, "The password should be at least {{length}} characters long, contain at least one number and have a mixture of uppercase and lowercase letters.")
control!.setClientValidationFunction(str(passwordValidator!))
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

## IsFalse

Validates that the control's value is false

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

falseValidator! = (new ValidationBuilder()).isFalse()
control!.setClientValidationFunction(str(falseValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |

## IsTrue

Validates that the control's value is true

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

trueValidator! = (new ValidationBuilder()).isTrue()
control!.setClientValidationFunction(str(trueValidator!))
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


## Match 

Validates that the control's value matches the value of a second control which can be found with the given selector.

### Usage

```BBj
use ::BBjControlValidation/ValidationBuilder.bbj::ValidationBuilder

confirmValidator! = (new ValidationBuilder()).match(".password-editbox", "The password does not match")
control!.setClientValidationFunction(str(confirmValidator!))
```

### Message Parameters

|  **Name**     |    **Description**   |
|  ---  |  ---  |
|   `x`, `value`, `text`     |    The control's value   |