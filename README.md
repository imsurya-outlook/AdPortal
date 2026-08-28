# Crossing Directory - MVP starter (ASP.NET Core MVC)

A working local advertising / Yellow Pages style portal for Crossing Republic: public discovery, advertiser
self-service, admin moderation, multi-image uploads, admin-verified businesses, user service ratings, and
view / details-click analytics. UI and API live in one MVC project so it deploys to a single Azure App Service.

## Run it (DEV)

    cd src/AdvertisingPortal.Web
    dotnet restore
    dotnet run

Then open the HTTPS URL shown in the console (default https://localhost:7189).

On first start the app creates the SQLite database (advertisingportal.db), seeds roles, the admin user,
the default locality Crossing Republic, and all 13 categories.

IMPORTANT after upgrading from an earlier build: the schema gained verified + rating columns and a ratings table.
Delete src/AdvertisingPortal.Web/advertisingportal.db once so it is recreated with the new shape.

### Seeded admin sign-in

    Email:    admin@localads.com
    Password: Admin@12345

Change these in appsettings.json (SeedAdmin section) before any real deployment.

### Try the full flow

1. Register a new account at /Account/Register - it becomes an Advertiser.
2. Post an advertisement with several photos and submit it for approval.
3. Sign in as the admin, open Admin -> Advertisements, approve it, then Verify it and optionally Feature it.
4. Browse the public site: featured ads sit in the sponsored band under Browse by category; verified ads carry a star.
5. Click a card (increments Details clicks) and load the detail page (increments Views).
6. Sign in as any other account and rate the business 1-5 stars with an optional comment.
7. Admin -> Ratings lists every rating; the advertiser dashboard shows the average per ad.

## Verified businesses

Only an admin can set the verified flag, from Admin -> Advertisements (Verify / Unverify). Verified ads show a
star badge on cards, in the title, and a note on the detail page. If an advertiser edits an approved ad and
resubmits it, the flag is cleared so the business is checked again.

## Service ratings

Signed-in users rate an approved ad from 1 to 5 stars with an optional comment. One rating per account per ad
(submitting again updates it), and owners cannot rate their own listing. Averages are stored on the ad as
RatingCount / RatingSum and recalculated on every write, so ordering and sorting stay fast.

## Ordering rules

    Recent ads      verified first, then highest average rating, then most ratings, then newest
    Search results  featured first, then verified, then highest rated, then newest
    Featured band   verified first, then highest rated, then newest

## Projects

- AdvertisingPortal.Domain - entities, enums, slug helper, seed constants
- AdvertisingPortal.Application - DTOs, service interfaces, upload limits, paging
- AdvertisingPortal.Infrastructure - EF Core DbContext, seeding, services, local + Azure Blob storage
- AdvertisingPortal.Web - MVC controllers, areas (Admin, Advertiser), Razor views, theme

## Key routes

    /                          home with hero search, categories, featured band, recent ads
    /ads                       search and filter results (locality defaults to Crossing Republic)
    /ads/{slug}                advertisement details (increments view count) + ratings
    /ads/{slug}/rate           submit or update a service rating (signed-in users)
    /ads/click/{id}            click tracker, redirects to the details page
    /category/{slug}           category listing
    /locality/{slug}           locality listing
    /Advertiser/Dashboard      advertiser stats, ratings and ad list
    /Advertiser/Advertisements advertiser CRUD, multi-image upload, cover image, submit
    /Admin/Dashboard           admin KPIs including verified count and average rating
    /Admin/Advertisements      moderation: approve, reject with remarks, verify, feature, activate
    /Admin/Ratings             every rating with reviewer, comment and remove action
    /Admin/Categories          add / edit / activate / remove categories
    /Admin/Localities          add / edit / activate / remove localities

## Configuration

Database provider is switched by configuration, so the same EF model runs on both engines:

    Database:Provider = Sqlite     (DEV, appsettings.Development.json)
    Database:Provider = SqlServer  (PROD, Azure SQL - appsettings.Production.json)

File storage is switched the same way behind IFileStorageService:

    Storage:Provider = Local       (DEV, writes to wwwroot/uploads)
    Storage:Provider = AzureBlob   (PROD, container advertisements)

Upload rules: max 10 images per advertisement, max 5 MB each, JPG / PNG / WebP only.

## Going to Azure

1. Create an Azure App Service (Windows or Linux, .NET 8) and an Azure SQL Database.
2. Create a storage account plus a blob container named advertisements.
3. In App Service configuration set: ConnectionStrings:DefaultConnection, Database:Provider = SqlServer,
   Storage:Provider = AzureBlob, Storage:ConnectionString, SeedAdmin:Email, SeedAdmin:Password.
4. Publish the AdvertisingPortal.Web project.

The starter calls EnsureCreated on startup so it runs with zero migration setup. When you want versioned schema
changes, generate the first migration and switch that call to Migrate:

    dotnet ef migrations add InitialCreate --project ../AdvertisingPortal.Infrastructure --startup-project .

## Deliberately left for later

Rating moderation workflow, lead / enquiry forms, map search, paid plans and checkout, SEO landing pages,
image resizing, and event-level analytics tables.
