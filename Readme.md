# New project
## Web asp.net core mvc + authen individual user account --> store in app
<img src="images/1.png">


## Build, run project, register user
<img src="images/2.png">





Case1:
CAN NOT - Edit Data > Migration > ApplicationDbContextModelSnapshot.cs
```
	modelBuilder.Entity("Microsoft.AspNetCore.Identity.IdentityUser", b =>
	                {
			…
			b.Property<DateTime>("DOB");
			b.Property<string>("HoTen").HasMaxLength(50);
			b.Property<string>("LinkNhaFB").HasMaxLength(256);
			…
```

### Add-Migration init
### Update-Database



Data > new class MyCustomUser.cs
```
 public class MyCustomUser : IdentityUser
    {
        [PersonalData]
        public string HoTen { get; set; }

        [PersonalData]
        public DateTime NgaySinh { get; set; }

        [PersonalData]
        public string LinKNhaFB { get; set; }
    }
```

ApplicationDbContext.cs
```
	public class ApplicationDbContext : IdentityDbContext<MyCustomUser>
	…
```



_LoginPartial.cshtml
```
	@inject SignInManager<WebApplication4.Data.MyCustomUser> SignInManager
	@inject UserManager<WebApplication4.Data.MyCustomUser> UserManager
```

Startup 
```
	ConfigureServices
		services.AddDefaultIdentity<MyCustomUser>()
```

### Add-Migration CustomUser1
### Update-Database
### Case1 remove 20190428161315_Init.cs
### Update-Database
<img src="images/3.png">

check DB table --> already have custom field



Run project
<img src="images/4.png">




Need to change Register Page to allow filled in custom field
Or After login, in manage page








Areas > add scrafold item > identity > 
	~/Views/Shared/_Layout.cshtml
	file Account\register (Optional) and Account\Manage\index
	ApplicationDbContext
<img src="images/5.png">


		
		
		

register pageModel (Optional)
```
  public class InputModel
	        {
	            [Required]
	            [DataType(DataType.Text)]
	            [Display(Name = "Tên Đầy Đủ")]
	            public string HoTen { get; set; }
	
	            [Required]
	            [Display(Name = "Ngày Sinh")]
	            [DataType(DataType.Date)]
	            public DateTime NgaySinh { get; set; }
	
	            [Required]
	            [DataType(DataType.Text)]
	            [Display(Name = "FB tường nhà")]
	            public string LinkNhaFB { get; set; }
		…
		
	OnPostAsync
		if (ModelState.IsValid)
	            {
	
		var user = new MyCustomUser {
		                    HoTen = Input.HoTen,
		                    NgaySinh = Input.NgaySinh,
		                    LinKNhaFB = Input.LinkNhaFB,
				…
```				

Register (Optional)
```
  <div class="form-group">
	                <label asp-for="Input.BeaName"></label>
	                <input asp-for="Input.BeaName" class="form-control" />
	                <span asp-validation-for="Input.BeaName" class="text-danger"></span>
	            </div>
	            <div class="form-group">
	                <label asp-for="Input.DOB"></label>
	                <input asp-for="Input.DOB" class="form-control" />
	                <span asp-validation-for="Input.DOB" class="text-danger"></span>
	            </div>
```


index pageModel
```
  public class InputModel
	        {
	            [Required]
	            [DataType(DataType.Text)]
	            [Display(Name = "Họ Tên Đầy đủ index")]
	            public string HoTen { get; set; }
	
	            [Required]
	            [Display(Name = "Ngày Tháng Năm Sinh")]
	            [DataType(DataType.Date)]
	            public DateTime NgaySinh { get; set; }
	
	            [Required]
	            [Display(Name = "FB tường nhà")]
	            [DataType(DataType.Text)]
	            public string LinkNhaFB { get; set; }
	
	OnGetAsync
		Input = new InputModel
		            {
		                HoTen = user.HoTen,
		                NgaySinh = user.NgaySinh,
		                LinkNhaFB = user.LinKNhaFB,
		
	OnPostAsync
		…
		if (Input.HoTen != user.HoTen)
		            {
		                user.HoTen = Input.HoTen;
		            }
		
		            if (Input.NgaySinh != user.NgaySinh)
		            {
		                user.NgaySinh = Input.NgaySinh;
		            }
		            if (Input.LinkNhaFB != user.LinKNhaFB)
		            {
		                user.LinKNhaFB = Input.LinkNhaFB;
		            }
		…
		await _userManager.UpdateAsync(user);
```


Index
```
 Lastest <div class="form-group">

                <div class="form-group">
                    <label asp-for="Input.HoTen"></label>
                    <input asp-for="Input.HoTen" class="form-control" />
                </div>
                <div class="form-group">
                    <label asp-for="Input.NgaySinh"></label>
                    <input asp-for="Input.NgaySinh" class="form-control" />
                </div>
                <div class="form-group">
                    <label asp-for="Input.LinkNhaFB"></label>
                    <input asp-for="Input.LinkNhaFB" class="form-control" />
                </div>
```

<img src="images/6.png">







### Disable REGISTER
- comment all code in page and pageModel
- ExternalLogin.cshtml
- ForgotPassword.cshtml
- ForgotPasswordConfirmation.cshtml
- Register.cshtml
- ResetPassword.cshtml
- ResetPasswordConfirmation.cshtml
- Pages > Account > Manage > ExternalLogin.cshtml


# Change MSSQL server to local SQLite db
nuget: `Microsoft.EntityFrameworkCore.Sqlite.Core`

Program.cs
```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));

builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite(connectionString));
```

appsettings.json
```json
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=aspnet-MoonWeb-1c125de0-dfab-4cfe-a0dc-2e6c61a87b23;Trusted_Connection=True;MultipleActiveResultSets=true"
  },

  "ConnectionStrings": {
    "DefaultConnection": "Filename=MyDatabase.db"
  },
```
Add-Migration init
Update-Database


# Add Role

Program.cs
```csharp
builder.Services.AddDefaultIdentity<IdentityUser>(options =>
{
    options.SignIn.RequireConfirmedAccount = true;
    options.User.RequireUniqueEmail = true;

})
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();

...

app.UseRouting();


using (var scope = app.Services.CreateScope())
{
    var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole>>();
    var roles = new[] { "Admin", "User" };
    foreach (var role in roles)
    {
        if (!await roleManager.RoleExistsAsync(role))
            await roleManager.CreateAsync(new IdentityRole(role));
    }
}
```

run project will create 2 rows in table `AspNetRoles`


# Model validation

Ref: https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation?view=aspnetcore-8.0

### Built-in attributes

Here are some of the built-in validation attributes:

- `[ValidateNever]`: Indicates that a property or parameter should be excluded from validation.
- `[CreditCard]`: Validates that the property has a credit card format. Requires jQuery Validation Additional Methods.
- `[Compare]`: Validates that two properties in a model match.
- `[EmailAddress]`: Validates that the property has an email format.
- `[Phone]`: Validates that the property has a telephone number format.
- `[Range]`: Validates that the property value falls within a specified range.
- `[RegularExpression]`: Validates that the property value matches a specified regular expression.
- `[Required]`: Validates that the field isn't null. See [Required] attribute for details about this attribute's behavior.
- `[StringLength]`: Validates that a string property value doesn't exceed a specified length limit.
- `[Url]`: Validates that the property has a URL format.
- `[Remote]`: Validates input on the client by calling an action method on the server. See [Remote] attribute for details about this attribute's behavior.


