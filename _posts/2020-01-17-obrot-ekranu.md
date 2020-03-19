---
date: 2020-01-17 10:26:40
layout: post
title: Zapisywanie stanu kontrolek w Androidzie
subtitle: Oto co się stanie gdy nie podasz id w layoutowym xml'u.
description: Kilka ważnych informacji o tym jak Android zapisuje stan kontolek np. przy obrocie ekranu.
image: https://res.cloudinary.com/dm7h7e8xj/image/upload/v1559825145/theme16_o0seet.jpg
optimized_image: https://res.cloudinary.com/dm7h7e8xj/image/upload/c_scale,w_380/v1559825145/theme16_o0seet.jpg
category: In depth
tags:
  - secret
  - protip
  - Android
author: Roman Mokrzan
---

Ciekawe jest to że niewiele osób wie dlaczego Androidowe kontrolki zachowują swój stan przy [zmianie konfiguracji, np. obracając ekran telefonu](https://developer.android.com/guide/topics/resources/runtime-changes).

Przykład: szybko tworzymy malutką aplikację z dwoma widokami [typu EditText](https://developer.android.com/reference/android/widget/EditText.html). Jeden z widoków będzie miał id a drugi nie, o tak:

```xml
<EditText
    android:id="@+id/password"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_marginStart="24dp"
    android:layout_marginTop="8dp"
    android:layout_marginEnd="24dp"

    android:hint="@string/prompt_password"
    android:imeActionLabel="@string/action_sign_in_short"
    android:imeOptions="actionDone"
    android:inputType="textPassword"
    android:selectAllOnFocus="true"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@+id/username" />

<!-- no android:id here ! -->
<EditText
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_marginStart="24dp"
    android:layout_marginTop="60dp"
    android:layout_marginEnd="24dp"

    android:hint="@string/prompt_password"
    android:imeActionLabel="@string/action_sign_in_short"
    android:imeOptions="actionDone"
    android:inputType="textPassword"
    android:selectAllOnFocus="true"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintHorizontal_bias="0.812"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintTop_toBottomOf="@+id/username" />
```

Co się stanie gdy odpalisz aplikację? Na pozór nic wielkiego. Ale teraz spróbuj wpisać coś do obu edittekstów. Wpisane? No to teraz obróć ekran. I co pan na to? Nie mamy pańskiego tekstu i co pan nam zrobi?

![placeholder](https://media.giphy.com/media/QynWLqh22Dx4sgzRbD/giphy.gif "Nie mamy pańskiego tekstu i co pan nam zrobi")

Jeśli ktoś **uważnie** studiował dokumentację Androida to znajdzie o tym małą wzmiankę w [Understand the Activity Lifecycle](https://developer.android.com/guide/components/activities/activity-lifecycle), tu pisze niech pan se przeczyta:

![placeholder](https://media.giphy.com/media/VJ5ZjC5Vugh9icKhoM/giphy.gif "Za kontrolki bez id Android nie odpowiada")

> Note: In order for the Android system to restore the state of the views in your activity, each view must have a unique ID, supplied by the android:id attribute.

Można to też znaleźć w samej dokumentacji kodu źródłowego [Androidowego Activity](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/core/java/android/app/Activity.java;l=2097?q=Activity):

```java
* <p>The default implementation takes care of most of the UI per-instance
* state for you by calling {@link android.view.View#onSaveInstanceState()} on each
* view in the hierarchy that has an id, and by saving the id of the currently
* focused view (all of which is restored by the default implementation of
* {@link #onRestoreInstanceState}).
```

I nieżej w [klasie View](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/core/java/android/view/View.java;l=20269) gdzie odbywa się **zapisywanie właściwe** na podstawie tego czy widok ma id czy nie ma:

```java
/**
 * Called by {@link #saveHierarchyState(android.util.SparseArray)} to store the state for
 * this view and its children. May be overridden to modify how freezing happens to a
 * view's children; for example, some views may want to not store state for their children.
 *
 * @param container The SparseArray in which to save the view's state.
 *
 * @see #dispatchRestoreInstanceState(android.util.SparseArray)
 * @see #saveHierarchyState(android.util.SparseArray)
 * @see #onSaveInstanceState()
 */
protected void dispatchSaveInstanceState(SparseArray<Parcelable> container) {
    if (mID != NO_ID && (mViewFlags & SAVE_DISABLED_MASK) == 0) {
        mPrivateFlags &= ~PFLAG_SAVE_STATE_CALLED;
        Parcelable state = onSaveInstanceState();
        if ((mPrivateFlags & PFLAG_SAVE_STATE_CALLED) == 0) {
            throw new IllegalStateException(
                    "Derived class did not call super.onSaveInstanceState()");
        }
        if (state != null) {
            // Log.i("View", "Freezing #" + Integer.toHexString(mID)
            // + ": " + state);
            container.put(mID, state);
        }
    }
}
```

Na dokładkę jeszcze należy pamiętać że jeśli piszesz swój własny customowy widok (czyli rozszerzasz klasę [View](https://developer.android.com/reference/android/view/View.html)) to najczęściej powinieneś nadpisać metodę [View.onSaveInstanceState()](https://developer.android.com/reference/android/view/View.html#onSaveInstanceState()). To właśnie ona zostanie zawołana w momencie gdy Android zawoła metodę [Activity.onSaveInstanceState](https://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)) na Aktywności w której ten customowy widok jest:

>>> The default implementation takes care of most of the UI per-instance state for you by calling View.onSaveInstanceState() on each view in the hierarchy that has an id, and by saving the id of the currently focused view (all of which is restored by the default implementation of onRestoreInstanceState(Bundle)). 

Jak myślisz, czy to jest dobre pytanie na rozmowe kwalifikacyjną na stanowisko Android developera? Dość podchwytliwe ale jeśli ktoś to wie to znaczy że dużo grzebał w dokumentacji / i bawił się z widokami. Co o tym sądzisz?
