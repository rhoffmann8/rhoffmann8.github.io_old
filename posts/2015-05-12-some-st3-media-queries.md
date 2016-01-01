So [Emmet](http://emmet.io) is pretty nice. Besides having tons of snippets out of the box, it allows for the addition of [custom snippets](http://docs.emmet.io/customization/snippets/) in either its settings file or in a **`snippets.json`** file in the extensions folder configured via the **`extensions_path`** setting in **`Emmet.sublime-settings`**.

I created a couple of media query snippets that maybe someone will find useful. If you want to utilize Bootstrap's Mobile First/Desktop First media queries but don't plan on using Bootstrap, typing **`@mfirst`** or **`@dfirst`** followed by tab will do that for you!

``` js
{
  "snippets": {
    "css": {
      "snippets": {
        "@mfirst": "@media only screen and (min-width: 320px) {\n\n}\n@media only screen and (min-width: 480px) {\n\n}\n@media only screen and (min-width: 768px) {\n\n}\n@media only screen and (min-width: 992px) {\n\n}\n@media only screen and (min-width: 1200px) {\n\n}\n",
        "@dfirst": "@media only screen and (max-width: 1200px) {\n\n}\n@media only screen and (max-width: 992px) {\n\n}\n@media only screen and (max-width: 768px) {\n\n}\n@media only screen and (max-width: 480px) {\n\n}\n@media only screen and (max-width: 320px) {\n\n}\n"
      }
    }
  }
}
```

@mfirst:

``` css
@media only screen and (min-width: 320px) {

}
@media only screen and (min-width: 480px) {

}
@media only screen and (min-width: 768px) {

}
@media only screen and (min-width: 992px) {

}
@media only screen and (min-width: 1200px) {

}
```

@dfirst:

``` css
@media only screen and (max-width: 1200px) {

}
@media only screen and (max-width: 992px) {

}
@media only screen and (max-width: 768px) {

}
@media only screen and (max-width: 480px) {

}
@media only screen and (max-width: 320px) {

}
```

Make sure your syntax is set to **CSS** for the trigger to work properly.