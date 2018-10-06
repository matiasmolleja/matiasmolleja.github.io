---
layout: post
title:  "Persist settings on Vue using localStorage"
date:   2018-09-30 09:37:33 +0200
categories: jekyll update
---
I think localStorage is perfect to persist some state between page reloads. I added this feature to the Assets-Media module on the awesome [Orchard Core CMS](https://github.com/OrchardCMS/OrchardCore) and it works pretty well. LocalStorage is pretty fast as it's local and has a [very good browser support](https://caniuse.com/#search=localstorage).

We are going to use a computed property to store the settings. And we will use the  `created` event to retrieve them on each page reload.

But first, let's create a simple app without any persistence feature.

#### HTML
```
<div id="app">

    <select v-model="activeColor">
        <option v-for="color in colors" v-bind:value="color">{{color}}</option>
    </select>

    <select v-model="activeNumber">
        <option v-for="number in numbers" v-bind:value="number">{{number}}</option>
    </select>

    <p>activeColor : {{activeColor}}</p>
    <p>activeColor : {{activeNumber}}</p>
</div>
```

#### JS
```
var app = new Vue({
    el: '#app',
    data: {
        colors: ['red', 'green', 'blue'],
        numbers: [1, 2, 3, 4],
        activeColor: 'red',
        activeNumber: 1
    }
});
```
If you try this code you can see that your activeColor and activeNumber values are lost when you reload the page. Let's fix that.

### Storing settings on each property change.
Computed properties are special vue properties. It's value depends on other properties. And any time any of thosehe  properties changes, Vue runs the function necessary to recompute the new value. This function is the perfect place to call localStorage.SetItem() and persist our values.

Let's add a paragraph to keep track of our "prefs" computed property:

#### HTML
```
<p>prefs : {{prefs}}</p>
```

Let's add our computed property after our data object:

#### JS
```
computed: {
    prefs: function () {
        var p = {
            storedColor: this.activeColor,
            storedNumber: this.activeNumber
        }
        localStorage.setItem('myDemoPrefs', JSON.stringify(p));
        return p;
    }
}
```
If you try this code you can see that any time you change one of the select tags, the stored item is updated with the new values (In chrome you can see that in action in Chrome Dev Tools: Application : Local Storage)

But if you reload the page, your settings are not keept, of course. We need to read them.

### Reading settings from local Storage.

Now we can use the `created` event to retrieve the stored settings and populate our active properties from them on each reload:

#### JS
```
created: function () {
    var loaded = JSON.parse(localStorage.getItem('myDemoPrefs'));
    if (loaded) {
        this.activeColor = loaded.storedColor;
        this.activeNumber = loaded.storedNumber;
    } else {
        console.warn('can\'t load preferences. Maybe your first time here?');
    }
}
```
Be careful: If you don't use your "prefs" computed property anywhere your new settings won't be saved. That is because Vue is so smart. It does not run the code to recalculate the property if the value is not going to be used anyway. If that is an issue, you can use a watcher instead.

I hope you find this technique useful.

I recorded a [video](https://youtu.be/3D0iK6B-P4k) too.
You can find the full thing on [my github profile](https://github.com/matiasmolleja/blog-posts-code/tree/master/blog-vue-localstorage).