# CoreMailer for NET Core 3.1 and above

Based on [Riyasat CoreMailer](https://www.nuget.org/packages/CoreMailer) project, this package is meant to work with NET Core 3.1 LTS and above. Send emails from dynamic templates made with Razor. 

**IMPORTANT** If you are using NET Core 3.1 and above **you must** configure your `Startup.cs` services to work with **controllers & views**, specially if the project was bootstraped to be a Razor Web App. Here we are using controllers & views to generate email views. More instructions below.


## How to Use

### Installation

Add package reference to your project `.csproj` file. For more installation methods refer to the package NuGet site.

    <PackageReference Include="Prosmart.CoreMailer" Version="1.0.0" />

### Usage

**In Startup.cs, ConfigureServices**

`TemplateRenderer` & `CoreMvcmailer` services must be cregistered as scoped services.

**REMEMBER** as per docs of NET Core 3.x, [MVC Registration](https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-3.1&tabs=visual-studio#mvc-service-registration) had some changes. If you are using a Razor Page project or WebAPI-only **you must register controllers & views** in order to use this package. So, the final service registration section would be like this:

    services.AddScoped<ITemplateRenderer, TemplateRenderer>();
    services.AddScoped<ICoreMvcMailer, CoreMvcMailer>();
    services.AddControllersWithViews();

You can use this MVC Registration alongside Razor Pages, read the docs provided to understand these changes.

Then, register routes to this controllers in Startup Configure section.

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });

This is pretty straightforward and basic configuration for **controllers**, you can change this configuration as you need. Official docs are available [here](https://docs.microsoft.com/en-us/aspnet/core/migration/22-to-30?view=aspnetcore-3.1&tabs=visual-studio#migrate-startupconfigure).


**In your Views folder**

Create a `cshtml` template under any views folder e.g.

    Views/Emails/Registration.cshtml

This template will be associated to a model so we can change values dynamically.

**The content of `cshtml` can be**
```csharp
    @model UserRegistrationInfo
    Hello <strong>@Model.UserName</strong> your email is <strong>@Model.Email</strong>
```

**NOTE: For emails you have to use inline styling.** This means that all CSS values must be inlined in your view file (or any HTML of some sort). For this task you can use [Foundation for Emails](https://foundation.zurb.com/emails/email-templates.html) or any web-based editor.

### Usage in Controllers
Emails must be _send_ inside a controller action, plain old MVC. Our mailer service is properly registered in our `Startup.cs` so we can use built-in DI in any controller constructor like this:

**Constructor**
```csharp
    private readonly ICoreMvcMailer _mailer;

    public HomeController(ICoreMvcMailer mailer)
    {
      _mailer = mailer;
    }
```


**In ActionMethod**
Now you can send emails inside any action, in this example we use the `SendAsync` method:


```csharp
    [AllowAnonymous]
    [HttpGet("users/notify")]
    public async Task<IActionResult> NotifyUser(string username, string email)
    {
        UserRegistrationInfo newUser = new UserRegistrationInfo()
        {
            UserName = username,
            Email = email 
        };

        MailerModel notifier = new MailerModel("YourHostName",1234)
        {
            FromAddress = "Your Address",
            IsHtml = true,
            User = "YourUserName",
            Key ="YourKey",
            ViewFile = "Emails/Register",
            Subject = "Registration",
            Model = newUser
        };
            
        await _mailer.SendAsync(notifier);
        return View();
    }
```    


**Local Folder Usage**

It is really simple to use. Just create MVCMailer model with **pickup directory location**. When you send the email make sure you set sender and reciver email. Once done, you can see email in your provided pickup directory.


```csharp
    MailerModel notifier = new MailerModel(**"Your Directory Here"**)
    {
        FromAddress = "Your Address",
        IsHtml = true,
        User = "YourUserName",
        Key ="YourKey",
        ViewFile = "Emails/Register",
        Subject = "Registration",
        Model = newUser
    };
    notifier.ToAddresses.Add("test@test.com");
    _mailer.Send(mdl);
```
