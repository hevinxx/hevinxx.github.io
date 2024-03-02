---
layout: post
title: "Tracking View Exposure in Android Apps Through Scrolling"
date: 2024-03-02 09:00:10 +0900
categories: 개발
tags: 안드로이드, 오픈소스
description: '"I want to track how much this banner has been exposed," but the developer said it would be difficult! Here is why and how to solve it.'
use_math: false
---
When trying to collect user activity data within an app, the first and most frequent thing you'll want to know is 'how much users have seen this view'. This is because once it's aggregated, it's possible to analyze how many users clicked, made a purchase, or left, among other things. Throughout my career working at various companies, the most common request I've received from data analysts has been to embed events to track how much a view has been exposed.

The first difficulty developers face when asked to embed view events is it's not clear when to mark them. Should you mark a view event again when the app is minimized by pressing the home button and then reopened? What about when a popup covers half the view and then is closed to reveal the view again? There are many ambiguous situations where it's unclear whether to mark a view event or not. Often, those requesting the feature haven't considered such various situations, leading developers to encounter these ambiguities while embedding the data events. Deciding on the spot whether to mark an event can lead to the worst-case scenario where each view event has different conditions defined. In the midst of this confusion, modifications to the view event trigger conditions might go unnoticed during feature updates.

However, this is not technically a problem. If it's clearly defined under what conditions a view event should be marked and it's well shared among stakeholders, then there shouldn't be an issue afterward. A bigger challenge is tracking the exposure of views within a scroll. Technically, tracking the exposure of views in a scroll is challenging.

To avoid the app feeling sluggish when a user scrolls, views must be created in advance before they appear on screen. For example, if a view containing an image is created only as it comes onto the screen, the user will have to wait for the image to load. If the view is pre-created and the image loaded beforehand, users can see the view smoothly as they scroll. Thus, the Android OS pre-creates views that will enter the scroll. Android app developers can execute desired logic at the timing of this view creation.

The issue is that this timing is for creating views, not exposing them. Because the next view is pre-created before scrolling, if the user doesn't scroll, the view may be created but not exposed. If view events are embedded at the time of creation, more view events may be marked than actual exposures.

You might think embedding view events at the time of exposure would solve the issue, but the Android OS doesn't provide a specific timing for exposure. Even the most skilled developers can't embed view events without a given timing. They must either accept the discrepancy and embed view events at the time of creation or calculate scroll heights and adjust for every possible edge case.

At the company I'm currently working for, we've chosen to accept the discrepancy. However, this year, as we started transitioning our app to use Jetpack Compose, we saw hope for improvement. Jetpack Compose allows view groups including scroll(such as LazyColumn, LazyRow) to track of views currently in the scroll. In other words, it's possible to determine which child views are exactly within the scroll as you move the Column or Row.

I won't detail the method here, as it's widely shared on the web, and this blog typically avoids posting code. However, it didn't take long to discover that many cases still couldn't be covered by this method.

![Ridi Capture](/assets/img/ridi.png)
For instance, in the case of nested scrolls, it's difficult to measure view events accurately. As an example, I've captured the RIDI app, which is mostly composed of a vertical scroll (Column) with a section called '웹툰 실시간 랭킹(Webtoon Real-Time Ranking)' composed of a horizontal scroll (Row). Within it, webtoons are grouped in threes vertically (Column). From the perspective of the Row determining whether to mark a view event, it positions the top 3 webtoons within its scroll. However, due to the scrolling condition of the Column, the 3rd webtoon is not fully exposed. Thus, while the Row might mark the view event for the 3rd webtoon, it's not fully exposed in reality.

Also, inserting or deleting views within a scroll is problematic. As mentioned, a Column or Row tracks the child views being displayed as it moves. However, if a view is inserted or deleted without any scroll movement, a previously visible view could move out of sight, or vice versa.

For this reason, we determined that the entity deciding if a view is exposed should not be the scrolling Column or Row, but the view itself that's the target of the view event. Fortunately, Jetpack Compose offers an onGloballyPositioned modifier, allowing execution of logic when a view's position changes. Thus, if a view's exposure can be determined through its position information, it's possible to know if a view is exposed without the help of a Column or Row.

However, determining a view's exposure based on its position information is not easy. onGloballyPositioned allows knowing a view's relative position within its parent view or its position in the entire window. Using the relative position within its parent doesn't indicate if the view is exposed because it's unclear where the parent view is located. Therefore, the position relative to the entire window must be used, which can reintroduce the issue with pre-created views outside the scroll area. If the scroll area only occupies the top half of the phone screen, pre-created views might be hidden in the bottom half, technically within the phone screen but not visible.

I wondered if this was a futile endeavor until I found a breakthrough. If the space occupied by the view, its parent view, and so on, all overlap to the same extent as the space the view occupies, then the view can be considered fully visible. The overlapped area with the parent view can only be equal to or smaller than the space the original view occupies. If it's smaller, it means the view isn't fully exposed and is only showing that smaller area. From the RIDI app capture, it's evident that the 3rd webtoon isn't fully exposed because the parent scroll view isn't completely visible, meaning the space it should occupy isn't fully secured. This approach allows for accurate determination of view exposure, regardless of how many scrolls are nested.

Starting with this idea, I created an open-source library called [Visibility Tracker]("https://github.com/hevinxx/visibility-tracker"). While developing it as an open-source library, I expanded it to measure not only whether a view is visible but also how much of it is visible in percentage. It also provides parameters to cover cases where the app is minimized with the home button and returned to, or when it's covered by another screen and returned via the back button.

However, there are limitations. For example, if a popup covers part of a view, tracking is impossible because the popup doesn't have a parent-child relationship with the view. I haven't found a solution for this yet. I hope someone else might find an answer, given it's an open-source library.