Android DOs: leverage the power of Data Binding adapters as a place for custom view logic
=======

>TL;DR There are cases when we need to write code which customizes or enhanced behavior of views. Data Binding adapters is a perfect place for that (instead of writing such code in activities, etc.), because they define a place for custom view logic and help to have a clear boundary between application code and framework code. Adapters is framework code.


Android Data Binding is truly a great library, not only does it make possible to use the ViewModel pattern on Android, but also makes great deal of help for Separation of Concerns.

Every developer needs to write custom view logic, from time to time: be it a custom class or just some glue code to define custom behavior between components, or within one component.

We used to write that type of code in activities or even Presenters, thus tightly coupling application code with framework code.

Data Binding adapters allow to move this view-specific code out from application logic, to a framework-specific side.

So, binding adapters are framework-specific and reside on the outer level of Clean Architecture.


We pick image view loading from url as an example.

It's a very trivial task, but for some reason loading images from URL hasn't been easy with Android, so you can't just set `android:src` to point to URL and expect it to be loaded there (and cached as well). But it would have been so nice to have!

A solution would be to introduce a custom binding adapter in which we'd implement image downloading code in a way we want. Say, we want to use Picasso.
And we also want to show image placeholder, for cases when it's being loaded or load failed, colored with a specific color.


```Kotlin
object ImageViewBindingAdapters {

    @BindingAdapter("src", "placeholder", "placeholderTint")
    @JvmStatic fun bindSrcUrl(imageView: ImageView, url: String, placeholder: Drawable, placeholderTint: Int) {
        val tintedPlaceholder = tintPlaceholder(placeholder, placeholderTint)
        val loadRequest = requestFromUrl(imageView.context, url)
        imageView.fitWithCenterCrop(loadRequest, tintedPlaceholder)
    }

    @BindingAdapter("src", "placeholderLetters", "placeholderTint")
    @JvmStatic fun bindSrcUrlPlaceholderLetters(imageView: ImageView, url: String, placeholderLetters: String, placeholderTint: Int) {
        val context = imageView.context
        val color = ContextCompat.getColor(context, placeholderTint)
        val letterPlaceholder = LetterDrawable(context, placeholderLetters, color)

        val loadRequest = requestFromUrl(imageView.context, url)
        imageView.fitWithCenterCrop(loadRequest, letterPlaceholder)
    }

    private @JvmStatic tintPlaceholder(placeholder: Drawable, tint: Int) =
            DrawableCompat.wrap(placeholder).apply {
                DrawableCompat.setTint(this, tint)
                DrawableCompat.setTintMode(this, PorterDuff.Mode.SRC_ATOP)
            }

    private @JvmStatic fun getUrlRequestCreator(context: Context, url: String?): RequestCreator {
        var url: String? = url
        if (url != null && (url.trim().isEmpty() || url.endsWith(".svg"))) {
            url = null
        }

        return Picasso.with(context).load(url)
    }
}
```

How do we use this? Just write

```XML
<ImageView
  ...
  android:src="some_url"/>
```

And you are done!

For example, you might want to change Picasso to use Glide instead, you can do that change in just one place, and this change will be automatically picked up by all image views.


## Benefits

#### Code is decouples

#### Binding adapters are easy to reuse throughout and across apps

#### Encapsulation of functionality


## Summary

In this article we've learned how Android Data Binding adapters help a lot for code separation and reusability: code in adapters is decoupled from your application logic, it is encapsulated well and can be reused across different applications.
