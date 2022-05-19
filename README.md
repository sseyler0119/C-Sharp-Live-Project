# C-Sharp-Live-Project

## Project background: 

This project is built using ASP.NET MVC and Entity Framework. I was part of a Scrum team for a two-week code sprint where we worked on an interactive website for managing the content and productions for a theater/acting company in Portland, Oregon called Theatre Vertigo. This site was intended to be a content management service for non-technical staff to manage their website content.  My team and I worked on the Production Area of the website where all productions, production photos, and cast member information is contained. 

## Front-End Stories
All front-end stories include the [color-palette](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/img/palette.png) provided by Theatre Vertigo and are designed to blend with the main portion of the website. Button styles, containers, and hover effects match those used in other areas of the website for consistency. 

### Style the About page (Main Site)
I was tasked with creating a new [About](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/Files/About.cshtml) page for Theatre Vertigo. I created a new view in the Home View folder for the About page and then used a mockup that was supplied by another UI/UX developer to recreate their design using HTML and [CSS](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/Files/Site.css). 

![About Page](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/img/About%20-%20My%20ASP.NET%20Application.gif)

## Production Area CRUD Pages:
### Style the Index Page

The specifications for styling the [Index](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/Files/Index.cshtml) required that the “Create New” link is changed to a button and the Cast Members are displayed using Bootstrap cards that are sorted by the Production they are currently acting in. Each card includes an overlay that dims the photo as the user hovers over the image and displays Font Awesome icons for the Edit and Delete pages. The image is wrapped in a link that directs to the Details page. 

### Style the Create and Edit Pages
For the [Create](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/Files/Create.cshtml) and [Edit](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/Files/Edit.cshtml)  pages, I added a header above the form, styled the “Submit” and “Back to List” buttons to match the website theme, included placeholders for each input field, and centered the form on the page. When each input field is clicked, the border color and background color changes to match the provided color palette. 

### Style the Delete and Details Pages
When styling the [Delete](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/Files/Delete.cshtml) and [Details](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/Files/Details.cshtml) pages, I centered the content and made the Cast Member’s photo, name, and bio more prominent within the container. Links to “Edit”, “Back to List”, and “Delete” were changed into buttons with corresponding Font Awesome Icons that matched the color palette and include hover effects. 

![Edit and Details](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/img/Edit%20and%20Details%20.gif)

![Delete](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/img/Delete%20-%20My%20ASP.NET%20Application.gif)

## Back-End Stories
### Create Entity Model and Scaffold the CRUD Pages
I created an entity model for the CastMember class so that cast members could be created and saved to the database. I modeled the CastMember based on a provided schema that specified which properties should be nullable, and I created a Position enum for the MainRole property. After creating the model, I used update-database to migrate the changes and create a table in the database. Then I scaffolded the CRUD pages using Visual Studio and Entity Framework with the existing DbContext and _Layout.cshtml page from the main part of the project. 
```cs

namespace TheatreCMS3.Areas.Prod.Models
{
    // Position enum for MainRole property
    public enum Position
    {
        Actor,
        Director,
        Technician,
        StageManager,
        Other
    }
    public class CastMember
    {
        public int CastMemberId { get; set; }
        public string Name { get; set; }
        public int? YearJoined { get; set; }
        public Position MainRole { get; set; }
        public string Bio { get; set; }
        public byte[] Photo { get; set; }
        public bool CurrentMember { get; set; }
        public string ProductionTitle { get; set; }
        public string Character { get; set; }
        public int? CastYearLeft { get; set; }
        public int? DebutYear { get; set; }
    }
}
```
![Database Table](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/img/DatabaseTable.png)

### Photo Storage and Retrieval

I modified the Create and Edit functions in the CastMember Controller to take in a parameter for an uploaded photo. This file is then passed to the UploadPhoto function where it is converted to a byte array. The RenderImage function converts the byte array back to an image and returns the image file to the web page to be displayed. 

```cs
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult Create([Bind(Include = "CastMemberId,Name,YearJoined,MainRole,Bio,Photo,CurrentMember,ProductionTitle, Character,CastYearLeft,DebutYear")] CastMember castMember, HttpPostedFileBase postedFile)
        {
            castMember.Photo = UploadPhoto(postedFile); // convert photo to byte[], bind to castMember.Photo
            if (ModelState.IsValid) // form is valid and file contains data
            {
                db.CastMembers.Add(castMember);
                db.SaveChanges();
                return RedirectToAction("Index");
            }

            return View(castMember);
        }
```

```cs
        // convert photo upload to byte array
        [HttpPost]
        public byte[] UploadPhoto(HttpPostedFileBase postedFile)
        {
            byte[] bytes;
            using(BinaryReader br = new BinaryReader(postedFile.InputStream))
            {
                bytes = br.ReadBytes(postedFile.ContentLength);
            }
            return bytes;
        }

        // convert byte[] back to image and return file to web page
        public ActionResult RenderImage(int id)
        {
            CastMember castMember = db.CastMembers.Find(id); // get id
            byte[] bytes = castMember.Photo; 
            return File(bytes, "image/jpeg"); 
        }
```

### Interactive Search Bar
For this story, I modified the [Index](#style-the-index-page) page to include an interactive search bar where the user can search Cast Members by their name or bio. As the user types, Cast Members that do not match any of the search terms are hidden from the view. This was accomplished by targeting the Id of the Search box and filtering the Cast Member cards and section dividers by class name using jQuery and the keyup event. 

![Index Page](https://github.com/sseyler0119/C-Sharp-Live-Project/blob/master/img/Index%20-%20Search%20Bar.gif)

```javascript
/************************ Cast Member Index Page ************************/

/* Search cast member name and bio for matching search term
 * As the user types, remove non-matches from view
 * Hide Production Title section headers when user starts to type
*/
$("#CastSearch").on("keyup", function () {
    var value = $(this).val().toLowerCase();
    $(".CastMember-Index--Card").filter(function () {
        $(this).toggle($(this).text().toLowerCase().indexOf(value) > -1)
    });
    $(".CastMember-Index--SectionDivider").filter(function () {
        $(this).toggle($(this).text().toLowerCase().indexOf(value) > -1)
    });
});

/************************ end Cast Member Index Page ************************/
```
