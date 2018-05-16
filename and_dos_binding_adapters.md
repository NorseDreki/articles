Android DOs: Data Binding adapters is a good place for custom view logic
=======

> TL;DR Custom view logic is present in almost any app, and Android Data Binding adapter is a better place for it rather than `Activity`, `Fragment`, etc., because it helps to decouple application code from framework glue code.

Android Data Binding is a truly great library: not only does it simplify use of the ViewModel pattern on Android, but also has ["binding adapters"](https://developer.android.com/topic/libraries/data-binding/binding-adapters), which allows to execute custom view logic around Android widgets.

Typically this type of logic is scattered across `Activity`s, `Fragment`s, etc. and also being duplicated here and there in a codebase.

Data Binding adapters allow to move this custom view logic out of application code, to the side of framework glue code. This results in separation of concerns, cleaner architecture, encapsulation of functionality, allows to keep changes (of custom view logic) in one place and draws a clear boundary between you app and the Android framework.

## Using Data Binding adapters by example of downloading image into `ImageView`

Even with help of great libraries such as Glide and Picasso, there is still some ceremony involved, before `android:src` of `ImageView` is populated.

A solution would be to introduce a binding adapter which implements image download, transformation and assignment logic.

In the example below, `bindSrcUrl` would download image from a URL, while providing placeholder for in progress and failed cases.

`bindSrcUrlPlaceholderLetters` would download image from a URL, but generates placeholder out of string with letters (this is for user avatars).

```Kotlin
object ImageViewBindingAdapters {

    @BindingAdapter("src", "placeholder", "placeholderTint")
    @JvmStatic fun bindSrcUrl(imageView: ImageView, url: String, placeholder: Drawable, placeholderTint: Int) {
        val tintedPlaceholder = tintPlaceholder(placeholder, placeholderTint)
        val loadRequest = requestFromUrl(imageView.context, url)

        //`fitWithCenterCrop` is an extension function loads and resizes image
        imageView.fitWithCenterCrop(loadRequest, tintedPlaceholder)
    }

    @BindingAdapter("src", "placeholderLetters", "placeholderTint")
    @JvmStatic fun bindSrcUrlPlaceholderLetters(imageView: ImageView, url: String, placeholderLetters: String, placeholderTint: Int) {
        val context = imageView.context
        val tint = ContextCompat.getColor(context, placeholderTint)
        val letterPlaceholder = LetterDrawable(context, placeholderLetters, tint)

        val loadRequest = requestFromUrl(imageView.context, url)
        imageView.fitWithCenterCrop(loadRequest, letterPlaceholder)
    }

    private @JvmStatic fun tintPlaceholder(placeholder: Drawable, tint: Int) =
            DrawableCompat.wrap(placeholder).apply {
                DrawableCompat.setTint(this, tint)
                DrawableCompat.setTintMode(this, PorterDuff.Mode.SRC_ATOP)
            }

    ...
}
```

Adapters can be used like so:

```XML
<ImageView
  ...
  bind:placeholder="@{@drawable/ic_item_black_36dp}"
  bind:placeholderTint="@{@color/secondaryAccent}"
  bind:src="@{viewModel.itemImageUrl}"/>
```

That's all -- now we can apply these adapters anywhere in application. Whenever there is a change in how image is downloaded, or new features such as image transformations are added, those would be applied centrally and wouldn't affect business logic of your application, because would reside on a side of framework glue code.

## Summary

In this article we've learned how Android Data Binding adapters help a lot for code separation and reusability: code in adapters is decoupled from your application logic, it is encapsulated well and can be reused across different applications.
