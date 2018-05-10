Android DOs: how to isolate framework by example of resources
=======

> TL;DR


```Kotlin
package com.upwork.android.mvvmp

import android.content.Context
import android.content.res.Resources.Theme
import android.support.annotation.*
import android.support.v4.content.res.ResourcesCompat

open class Resources(
    private val appContext: Context
) {
    open fun getString(@StringRes id: Int, vararg formatArgs: Any) =
        appContext.resources.getString(id, *formatArgs)!!

    open fun getDrawable(@DrawableRes id: Int, theme: Theme? = null) =
        ResourcesCompat.getDrawable(appContext.resources, id, theme)!!

    open fun getColor(@ColorRes id: Int, theme: Theme? = null) =
        ResourcesCompat.getColor(appContext.resources, id, theme)

    open fun getQuantityString(@PluralsRes id: Int, quantity: Int, vararg formatArgs: Any) =
        appContext.resources.getQuantityString(id, quantity, *formatArgs)!!

    fun getInteger(@IntegerRes id: Int) = appContext.resources.getInteger(id)
}
```
