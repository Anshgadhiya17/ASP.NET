# ASP.NET Core File Upload Demo
## Code Overview

### Model: UserModal.cs
```csharp
namespace Praticse.Models
{
    public class UserModal
    {
        public IFormFile profileImg { get; set; }
        public string name;
    }
}

```


### Controller: HomeController.cs

```csharp
using System.Diagnostics;
using System.Net.Sockets;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Praticse.Helpers;
using Praticse.Models;

namespace Praticse.Controllers;

public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;
    public static List<UserModal> users = new List<UserModal>();
    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }
    public void updateUser(UserModal user)
    {
        int idx = users.FindIndex(u=>u.id==user.id);
        users[idx].name = user.name;
        users[idx].profileImgSrc = user.profileImgSrc;
        // userGlobal = user;
    }
    public UserModal getUser(int id){
        UserModal user = users.Find(u=>u.id==id) ?? new UserModal();
        return user;
    }
    public void insertUser(UserModal user){
        user.id = (new Random()).Next()/1000;
        Console.WriteLine(user.id);
        users.Add(user);
    }

    public IActionResult Index()
    {
        return View(users);
    }

    

    public IActionResult AddEdit(int? id)
    {
        UserModal user = new UserModal();
        if(id != null && id != 0){
            user = getUser((int)id);
            Console.WriteLine(user.name);
        }
        return View(user);
    }

    public IActionResult Save(UserModal user)
    {
        Console.WriteLine(user.name);
        if (user.id == null)
        {
            try
            {
                user.profileImgSrc = ImageHelper.SaveImage(user.profileImg, "Profile");
            }
            catch (System.Exception)
            {
                Console.WriteLine("file not provided mostly");
            }  
            insertUser(user);
        }
        else
        {
            if (user.profileImg != null)
            {
                ImageHelper.DeleteFileFromUrl(user.profileImgSrc??"");
                user.profileImgSrc = ImageHelper.SaveImage(user.profileImg, "Profile");
            }
            updateUser(user);
        }


        return RedirectToAction("Index");
    }

    

    public IActionResult Privacy()
    {
        return View();
    }

    [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
    public IActionResult Error()
    {
        return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
    }
}

```

### ImageHelper.cs

```csharp
namespace Praticse.Helpers;

using System;
using System.IO;
public class ImageHelper
{
    public static string SaveImage(IFormFile imageFile, string dir)
    {
        string finalDirPath = $"wwwroot/{dir}";
        if (imageFile == null || imageFile.Length == 0)
        {
            throw new Exception();
        }

        if (!Directory.Exists(finalDirPath))
        {
            Directory.CreateDirectory(finalDirPath);
        }
        //extract extension from file
        string fileExtension = Path.GetExtension(imageFile.FileName);

        //genrate unique file name
        string uniqueNameForFile = $"{Guid.NewGuid()}.{fileExtension}";

        //get full path which we will store in db ( we dont need to store from wwwroot)
        string fullPathToStoreInDB = $"{dir}/{uniqueNameForFile}";

        //get path where we store image means wwwroot
        string fullPathToWrite = $"{finalDirPath}/{uniqueNameForFile}";


        // use stream to manipulate or save image in disk
        FileStream stream = new FileStream(fullPathToWrite, FileMode.CreateNew);
        imageFile.CopyTo(stream);
        stream.Close();

        //retrurn path which we will store in db
        return fullPathToStoreInDB;

    }
    public static string DeleteFileFromUrl(string filePath)
    {
        // Validates that the file path is not empty
        if (string.IsNullOrWhiteSpace(filePath))
            return "File URL is required.";

        // Constructs the physical path to the file in the wwwroot/filepath directory
        var finalFilePath = Path.Combine(Directory.GetCurrentDirectory(), "wwwroot", filePath);
        Console.WriteLine(finalFilePath);
        // Checks if the file exists
        if (!System.IO.File.Exists(finalFilePath))
            return "File not found.";

        try
        {
            // Deletes the file from the file system
            System.IO.File.Delete(finalFilePath);
            return "File deleted successfully.";
        }
        catch (Exception ex)
        {
            // Rethrows the exception to be handled by the caller
            throw;
        }
    }
}
```

### View: Index.cshtml

- Upload form with `multipart/form-data`.

```csharp
<div class="text-center">
    <h1 class="display-4">Welcome</h1>
    <p>Learn about <a href="https://learn.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>

    <form enctype="multipart/form-data" method="post" asp-action="Save" asp-controller="Home">
        <input name="name" />
        <input type="file" name="profileImg" />
        <input type="submit" />
    </form>
</div>
```
