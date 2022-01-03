# plugin-banner
A [Micro.blog](https://micro.blog "Micro.blog") plugin that makes it easy to drop page-specific banners into your theme.

WARNING: This README has not been updated yet for version 3.0.0.

![Banner 1](https://raw.githubusercontent.com/moonbuck/plugin-banner/main/images/banner1.jpeg)

![Banner 2](https://raw.githubusercontent.com/moonbuck/plugin-banner/main/images/banner2.jpeg)

![Banner 3](https://raw.githubusercontent.com/moonbuck/plugin-banner/main/images/banner3.jpeg)

The lone template lives at `layouts/partials/plugin_banner.html` and looks like this:

```go
{{ $banner_data := false }}
{{ with site.Data.plugin_banner.banner_data }}
{{ $banner_data = . }}
{{ else }}
{{ with site.Params.banner_data }}{{ $banner_data = . }}{{ end }}
{{ end }}

{{ if (and $banner_data (not ($banner_data | reflect.IsMap))) }}
{{ with transform.Unmarshal $banner_data }}
{{ $banner_data = . }}
{{ end }}
{{ end }}

{{ if $banner_data }}
{{- $path := (urls.Parse .Permalink).Path -}}
{{- with (index $banner_data $path) -}}
<div id="{{ anchorize $path }}" class="banner-image" style="background-image: url({{ . }});"></div>
{{- end -}}
{{ end }}
```

The banner is applied as a CSS background image. It gets a `banner-image` class slapped onto it and an anchorized version of the page location set as its `id`.

The base style is applied by the tiny bit of CSS living at `static/assets/css/plugin_banner.css`. It can be manipulated to your heart’s content and looks like this out of the box:

```css
.banner-image {
  background-color: #516189;
  height: 300px;
  width: 100%;
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  position: relative;
}
```

Images. It’s gonna need images. However will it find them?

Option one: drop a JSON object in the plugin parameter field with the banner data.

![Plugin Parameters](https://raw.githubusercontent.com/moonbuck/plugin-banner/main/images/plugin_parameters.jpeg)

Option two (and my new favorite): create a JSON file living at `data/plugin_banner/banner_data.json`.

![File Location](https://raw.githubusercontent.com/moonbuck/plugin-banner/main/images/file_location.jpeg)

My file happens to look like  this:

```json
{
  "/": "https://moondeer.blog/uploads/2021/8135337116.png",
  "/about/": "https://moondeer.blog/uploads/2021/955619b235.jpg",
  "/cloud/": "https://moondeer.blog/uploads/2021/547d825d8a.jpg",
  "/bookshelf/": "https://moondeer.blog/uploads/2021/27a279361f.jpg",
  "/gallery/":"https://moondeer.blog/uploads/2021/8585a4a081.jpg",
  "/categories/perspectives/": "https://moondeer.blog/uploads/2021/f5f64b49bb.jpg",
  "/categories/projects/": "https://moondeer.blog/uploads/2021/74512379fa.png",
  "/categories/poetry/": "https://moondeer.blog/uploads/2021/02bec3693a.png",
  "/categories/music/": "https://moondeer.blog/uploads/2021/b604ce0519.png",
  "/categories/programming/": "https://moondeer.blog/uploads/2021/0d2564a5e5.png",
  "/categories/critters/": "https://moondeer.blog/uploads/2021/e0dfd20403.png",
  "/categories/stream-of-consciousness/": "https://moondeer.blog/uploads/2021/b48fce578f.png",
  "/categories/inside-the-art/": "https://moondeer.blog/uploads/2021/8c4669346c.jpg",
  "/categories/biographical-tripe/": "https://moondeer.blog/uploads/2021/ac4a187662.png",
  "/categories/artsy-fartsy/": "https://moondeer.blog/uploads/2021/608bfd1756.png",
  "/categories/personal-favorites/": "https://moondeer.blog/uploads/2021/7529b05b3e.png"
}
```

The keys are the page paths once you remove `[SCHEME]://HOSTNAME` from them. The values are the images you want to use.

The `id` values attached to those `<div>` tags are gonna match the last bit of that path (except for the home page, which will have `home` slapped on it). This lets you fine tune your banners, kinda like…

```css
div#categories-stream-of-consciousness 
  background-position: 50% 5%;
}

/* Media Queries */

@media screen and (max-width: 540px) {
  div#categories-artsy-fartsy {
     background-position: 35% 50%;
  }
  div#categories-critters {
     background-position: 35% 50%;
  }
}

```

Okay, you’re sold. But how do we get these things to show up? Simple. Pick a spot or two in your theme (I chose `layouts/_default/baseof.html` and `layouts/index.html`) and drop in this bit of code:

```go
{{ if templates.Exists "partials/plugin-banner.html" }}
{{ partial "plugin-banner.html" . }}
{{ end }}
```
