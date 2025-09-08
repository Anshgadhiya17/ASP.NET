# Custom Tag Helper

Tag Helpers allow us to **extend or replace HTML tags** in Razor views using server-side C# code.  
They make HTML cleaner, reusable, and easier to maintain.

---

## Example 1: MailToTagHelper

👉 Goal: Convert a `<mail>` tag into an `<a>` tag (email link).
## Step 1: Create a `MailToTagHelper` Class
### Code:
```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;

namespace MyApp.TagHelpers
{
    [HtmlTargetElement("mail")]  // This helper works on <mail> tags
    public class MailToTagHelper : TagHelper
    {
        public string Address { get; set; }  // Attribute: address

        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            output.TagName = "a";  // Replace <mail> with <a>
            output.Attributes.SetAttribute("href", "mailto:" + Address);
            output.Content.SetContent("Contact Us");
        }
    }
}
````
## Step 2: Register the Tag Helper

👉 Razor needs to know where your TagHelpers live.

Open `_ViewImports.cshtml` (inside `Views`) and add:

```razor
@addTagHelper *, namespaceOfCustomTagHelper   <!-- Replace MyApp with your project name -->
```
## Step 3: Usage (Index.cshtml):

```razor
<mail address="support@myapp.com"></mail>
```

### Output (Browser HTML):

```html
<a href="mailto:support@myapp.com">Contact Us</a>
```

**Explanation:**

* `[HtmlTargetElement("mail")]` → This helper only works on `<mail>`.
* `output.TagName = "a";` → Final HTML becomes `<a>`.
* `output.Attributes.SetAttribute("href", ...)` → Adds an `href` attribute.
* `output.Content.SetContent("Contact Us");` → Sets the inner text.

---

## Example 2: DateTagHelper

👉 Goal: Convert `<date>` tag into today’s date.

### Code:

```csharp
using System;
using Microsoft.AspNetCore.Razor.TagHelpers;

namespace MyApp.TagHelpers
{
    [HtmlTargetElement("date")]  // This helper works on <date> tags
    public class DateTagHelper : TagHelper
    {
        public string Format { get; set; } = "dd-MM-yyyy"; // Default format

        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            output.TagName = "span";  // Replace <date> with <span>
            string today = DateTime.Now.ToString(Format);
            output.Content.SetContent(today);
        }
    }
}
```

### Usage (Index.cshtml):

```razor
<p>Default format: <date></date></p>
<p>Custom format: <date format="MMMM dd, yyyy"></date></p>
```

### Output (Browser HTML):

```html
<p>Default format: <span>08-09-2025</span></p>
<p>Custom format: <span>September 08, 2025</span></p>
```

**Explanation:**

* `[HtmlTargetElement("date")]` → This helper only works on `<date>`.
* `Format` property → Value comes from the `format` attribute.
* `output.TagName = "span";` → Final HTML uses `<span>`.
* `output.Content.SetContent(today);` → Inserts today’s date.

---

## Steps to Use Any Tag Helper

1. **Create a class** inside a `TagHelpers/` folder.
2. **Inherit** from `TagHelper`.
3. **Override** the `Process` method.
4. **Register** in `_ViewImports.cshtml`:

   ```razor
   @addTagHelper *, MyApp
   ```
5. **Use** the new tag in your Razor view.
6. **Run & Inspect** the output in the browser.

---

