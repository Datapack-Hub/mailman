![](https://cdn.discordapp.com/attachments/1129493191847071875/1147217243751583865/image.png)

# Mailman

Mailman is a tool which lets you upload your datapack versions to Datapack Hub, Modrinth, and Smithed at the same time.

Upload your datapack, enter your API tokens, add version details, and then push your datapack to all sites at once.

**Website**: https://mailman.datapackhub.net/

## How to use

The website is designed to be very easy to use if you are familiar with uploading versions on other sites.

1. Upload your datapack (you can drag-and-drop it onto the site)
2. Enter your API tokens for DPH, Modrinth, and Smithed.
3. Pick the project that you want to add the version to.
4. Enter the relevant details (version title, code, changelog, minecraft versions, dependencies)
5. Hit Upload, and you're done!

## The code part
Pretty much *all* the code (HTML, JS) is in /src/routes/+page.svelte. 