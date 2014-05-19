---
layout: post
title: Associated Objects
framework: "Objective-c"
rating: 6.6
description: "연결객체는 Objective-C 2.0 실행환경의 기능으로 실행 중에 한 객체에 어떤 임의의 객체를 키-값으로 연결할 수 있다. 다크 주주처럼 objc/runtime.h에 있는 다른 함수들 처럼 매우 조심스럽게 다루어야 한다.  Associated Objects is a feature of the Objective-C 2.0 runtime, which allows objects to associate arbitrary values for keys at runtime. It's dark juju, to be handled with as much caution as any other function from objc/runtime.h"
---

~~~{objective-c}
#import <objc/runtime.h>
~~~

>Objective-C developers are conditioned to be wary of whatever follows this ominous incantation. And for good reason: messing with the Objective-C runtime changes the very fabric of reality for all of the code that runs on it.

오브젝티브-씨 개발자라면 이 불길한 주술에 무언가 딸려올 것 같은 불안감에 길들여진다. 이유를 보자면, 실행환경 위에서 실행되는 모든 코드를 실체의 근본을 변경시켜 Objective-C 실행 환경을 망쳐버리는 것이 있다.

>In the right hands, the functions of `<objc/runtime.h>` have the potential to add powerful new behavior to an application or framework, in ways that would otherwise not be possible. In the wrong hands, it drains the proverbial [sanity meter](http://en.wikipedia.org/wiki/Eternal_Darkness:_Sanity's_Requiem#Sanity_effects) of the code, and everything it may interact with (with [terrifying side-effects](http://www.youtube.com/watch?v=RSXcajQnasc#t=0m30s)).

좋은 방향으로는, 잘 안될 수도 있겠지만, `<objc/runtime.h>`의 함수들은 애플리케이션이나 프레임워크에 강력한 새 행위를 넣을 잠재력을 가지고 있다. 안좋은 방향으로는, 코드에서 그 유명한 정신력이란 것을 소모하고, 모든 것([무시무시한 부수효과](http://www.youtube.com/watch?v=RSXcajQnasc#t=0m30s) 를 포함해서)과 상호작용할 수 있다.

>Therefore, it is with great trepidation that we consider this [Faustian bargain](http://en.wikipedia.org/wiki/Deal_with_the_Devil), and look at one of the subjects most-often requested by NSHipster readers: associated objects.

이래서 우리는 엄청난 혼동 속에서 이 [악마의 계약](http://en.wikipedia.org/wiki/Deal_with_the_Devil)을 고려한다, 그래서 NSHipster 독자가 가장 자주 많이 요청하는 주제를 살펴보게 되었다. 바로  associated objects 이다.

* * *

>Associated Objects—or Associative References, as they were originally known—are a feature of the Objective-C 2.0 runtime, introduced in Mac OS X 10.6 Snow Leopard (available in iOS 4). The term refers to the following three C functions declared in `<objc/runtime.h>`, which allow objects to associate arbitrary values for keys at runtime:

연관 객체(원래 연관 참조로 알려져 있다.)는 Objective-C 2.0 실행환경 기능이다. 이는 Mac OS X 10.6 스노우 레오파드 와 iOS4에서 소개되었다. 용어는 실행 중 객체에 키에 따라 임의의 값을 연결시켜주는 `<objc/runtime.h>` 에 정의된 3개의 C 함수에서 따왔다. 

- `objc_setAssociatedObject`
- `objc_getAssociatedObject`
- `objc_removeAssociatedObjects`

>Why is this useful? It allows developers to **add custom properties to existing classes in categories**, which [is an otherwise notable shortcoming for Objective-C](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html).

이것들이 왜 유용할까? [Objective-C에 이미 존재하는 유명한 결점](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html)이 있는 **기존 클래스에 카테고리로 커스텀 속성을 추가** 할 수 있다.


#### NSObject+AssociatedObject.h

~~~{objective-c}
@interface NSObject (AssociatedObject)
@property (nonatomic, strong) id associatedObject;
@end
~~~

#### NSObject+AssociatedObject.m

~~~{objective-c}
@implementation NSObject (AssociatedObject)
@dynamic associatedObject;

- (void)setAssociatedObject:(id)object {
     objc_setAssociatedObject(self, @selector(associatedObject), object, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

- (id)associatedObject {
    return objc_getAssociatedObject(self, @selector(associatedObject));
}
~~~

>It is often recommended that they key be a `static char`—or better yet, the pointer to one. Basically, an arbitrary value that is guaranteed to be constant, unique, and scoped for use within getters and setters:

키는 `static char`형으로 추천하는데 혹은 포인터가 더 좋은 것 같다. 기본적으로 임의의 어떤 값이 게터와 세터안에서 사용될 때 상수성, 유일성, 범위성을 보장하게 된다.

~~~{objective-c}
static char kAssociatedObjectKey;

objc_getAssociatedObject(self, &kAssociatedObjectKey);
~~~

>However, a much simpler solution exists: just use a selector.

그러나 훨씬 더 간단한 해법이 존재한다. 그냥 셀렉터를 써라.

<blockquote class="twitter-tweet" lang="en"><p>Since <tt>SEL</tt>s are guaranteed to be unique and constant, you can use <tt>_cmd</tt> as the key for <tt>objc_setAssociatedObject()</tt>. <a href="https://twitter.com/search?q=%23objective&amp;src=hash">#objective</a>-c <a href="https://twitter.com/search?q=%23snowleopard&amp;src=hash">#snowleopard</a></p>&mdash; Bill Bumgarner (@bbum) <a href="https://twitter.com/bbum/statuses/3609098005">August 28, 2009</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

>## Associative Object Behaviors

## 연관객체 행위

>Values can be associated onto objects according to the behaviors defined by the enumerated type `objc_AssociationPolicy`:

값들은 `objc_AssociationPolicy`에 나열된 타입에 정의된 행동에 따라 객체와 연결되게 된다:

<table>
    <thead>
        <tr>
            <th>Behavior</th>
            <th><tt>@property</tt> Equivalent</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <tt>OBJC_ASSOCIATION_ASSIGN</tt>
            </td>
            <td>
                <tt>@property (assign)</tt> or <tt>@property (unsafe_unretained)</tt>
            </td>
            <td>
                Specifies a weak reference to the associated object.
				연결 객체가 약한 참조로 설정 된다.
            </td>
        </tr>
        <tr>
            <td>
                <tt>OBJC_ASSOCIATION_RETAIN_NONATOMIC</tt>
            </td>
            <td>
                <tt>@property (nonatomic, strong)</tt>
            </td>
            <td>
                Specifies a strong reference to the associated object, and that the association is not made atomically.
				연결 객체가 강한 참조로 설정 되며 원자적으로 연결되지 않는다.
            </td>
        </tr>
        <tr>
            <td>
                <tt>OBJC_ASSOCIATION_COPY_NONATOMIC</tt>
            </td>
            <td>
                <tt>@property (nonatomic, copy)</tt>
            </td>
            <td>
                Specifies that the associated object is copied, and that the association is not made atomically.
				연결 객체가 복사로 설정되며 원자적으로 연결되지 않는다.
            </td>
        </tr>
        <tr>
            <td>
                <tt>OBJC_ASSOCIATION_RETAIN</tt>
            </td>
            <td>
                <tt>@property (atomic, strong)</tt>
            </td>
            <td>
                Specifies a strong reference to the associated object, and that the association is made atomically.
				연결 객체가 강한 참조로 설정이 되면서 원자적으로 연결된다.
            </td>
        </tr>
        <tr>
            <td>
                <tt>OBJC_ASSOCIATION_COPY</tt>
            </td>
            <td>
                <tt>@property (atomic, copy)</tt>
            </td>
            <td>
                Specifies that the associated object is copied, and that the association is made atomically.
				연결 객체가 복사되면서 원자적으로 연결된다.
            </td>
        </tr>
    </tbody>
</table>

>Weak associations to objects made with `OBJC_ASSOCIATION_ASSIGN` are not zero `weak` references, but rather follow a behavior similar to `unsafe_unretained`, which means that one should be cautious when accessing weakly associated objects within an implementation.

`OBJC_ASSOCIATION_ASSIGN`으로 만들어진 객체의 약한 관계는 `weak` 참조가 아닌뿐만 아니라 오히려`unsafe_unretained`와 유사한 동작을 하도록 되어 있다. 즉 구현체 내부에서 약하게 연결된 객체를 접근하고자 할 떄 반드시 주의해야한다는 것을 의미한다.


> According to the Deallocation Timeline described in [WWDC 2011, Session 322](https://developer.apple.com/videos/wwdc/2011/#322-video) (~36:00), associated objects are erased surprisingly late in the object lifecycle, in `object_dispose()`, which is invoked by `NSObject -dealloc`.

[WWDC 2011, Session 322](https://developer.apple.com/videos/wwdc/2011/#322-video)에 설명한 객체 해제 타임라인에 따르면 연결 객체는 객체 생명주기 이후 `NSObject -dealloc`에서 호출되는 `object_dispose()`에서 갑자기 지워질 수 있다. 

>## Removing Values

## 값지우기


>One may be tempted to call `objc_removeAssociatedObjects()` at some point in their foray into associated objects. However, [as described in the documentation](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_removeAssociatedObjects), it's unlikely that you would have an occasion to invoke it yourself:

어느 객체가 어떤 시점에 자신이 관여하고 있는 연결 객체를 가지고 `objc_removeAssociatedObjects()`를 호출하려고 할지도 모른다. 그러나 [문서에 기술되어 있는 것처럼](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html#//apple_ref/c/func/objc_removeAssociatedObjects) 자기 자신이 자신을 호출하려고 하는 상황이 생길 수 도 있다.

>The main purpose of this function is to make it easy to return an object to a "pristine state”. You should not use this function for general removal of associations from objects, since it also removes associations that other clients may have added to the object. Typically you should use objc_setAssociatedObject with a nil value to clear an association.

이 함수의 주요 기능은 객체가 "순수 상태"로 손쉽게 반환되도록 해주는 것이다. 특정 객체에 다른 객체들이 추가한 연관 객체까지 모두 제거해버리기 때문에 특정 객체에서 연관 객체의 범용적 삭제를 위한 함수로 사용해서는 안될 것이다. 

## Patterns

>- **Adding private variables to facilitate implementation details**. When extending the behavior of a built-in class, it may be necessary to keep track of additional state. This is the _textbook_ use case for associated objects. For example, AFNetworking uses associated objects on its `UIImageView` category to [store a request operation object](https://github.com/AFNetworking/AFNetworking/blob/2.1.0/UIKit%2BAFNetworking/UIImageView%2BAFNetworking.m#L57-L63), used to asynchronously fetch a remote image at a particular URL.

- **실제 구현을 손쉽게 해주기 위한 프라이빗 변수 추가**. 고유 클래스 행위를 확장하려고 할 때, 추가 상태 추적을 유지할 필요가 있을 수도 있다. 이것이 연관 객체의 _교과서_ 적인 사용 예이다. 예를 들면, AFNetworking 은 `UIImageView` 카테고리에 연관 객체를 사용해 특정 URL에서 원격 이미지를 비동기로 가져오는 [요청 오퍼레이션 객체를 저장](https://github.com/AFNetworking/AFNetworking/blob/2.1.0/UIKit%2BAFNetworking/UIImageView%2BAFNetworking.m#L57-L63)하는데 사용한다.

- **Adding public properties to configure category behavior.** Sometimes, it makes more sense to make category behavior more flexible with a property, than in a method parameter. In these situations, a public-facing property is an acceptable situation to use associated objects. To go back to the previous example of AFNetworking, its category on `UIImageView`, [its `imageResponseSerializer`](https://github.com/AFNetworking/AFNetworking/blob/2.1.0/UIKit%2BAFNetworking/UIImageView%2BAFNetworking.h#L60-L65) allows image views to optionally apply a filter, or otherwise change the rendering of a remote image before it is set and cached to disk.

>- **카테고리 행동을 조정하는데 사용하는 공개 속성을 추가** 가끔식, 메소드 인자보다 카테고리 행위를 속성을 사용해서 좀 더 유연하게 만들 필요성을 느낄 때가 있다. 이런 상황에서는 공개된 속성은 연관 객체를 사용해서 이런 상황을 해소해줄 수 있다. 좀 전에 본 AFNetworking 예를 다시 보면,  `UIImageView` 카테고리에는 [`imageResponseSerializer` 이라는](https://github.com/AFNetworking/AFNetworking/blob/2.1.0/UIKit%2BAFNetworking/UIImageView%2BAFNetworking.h#L60-L65) 속성이 있는데 이미지뷰에 선택적으로 필터를 적용하거나 설정하고 설정되거나 디스크에 캐시되기 전에 원객 이미지를 변경할 수 있다.

>- **Creating an associated observer for KVO**. When using [KVO](http://nshipster.com/key-value-observing/) in a category implementation, it is recommended that a custom associated-object be used as an observer, rather than the object observing itself.

- **KVO에 연결된 옵저버를 만들기** 카테고리 구현에서 [KVO](http://nshipster.com/key-value-observing/)를 쓸 때 관찰될 객체 자신보다 옵저버에서 사용될 커스텀 연관 객체를 사용하는 것을 추천한다.

-## Anti-Patterns
## 안티패턴 - 이렇게는 쓰지말자

>- **Storing an associated object, when the value is not needed**. A common pattern for views is to create a convenience method that populates fields and attributes based on a model object or compound value. If that value does not need to be recalled later, it is acceptable, and indeed preferable, not to associate with that object.

- **값이 필요 없는데도 연관 객체를 저장하기**. 뷰를 사용하는 일반적인 패턴은 모델 객체나 합성값에 따른 필드와 선언하는 간편한 메소드를 만드는 것이다. 만약 나중에 다시 호출할 필요가 없는 값이라고 하면 받을 수 있게 만들거나 진짜 더 나은쪽은 객체에 연관 객체를 사용하지 안흔ㄴ 것이다.

>- **Storing an associated object, when the value can be inferred.** For example, one might be tempted to store a reference to a custom accessory view's containing `UITableViewCell`, for use in `tableView:accessoryButtonTappedForRowWithIndexPath:`, when this can retrieved by calling `cellForRowAtIndexPath:`.

- **추정할 수 있는 값을 연관 객체에 저장하기** 예를 들면,`tableView:accessoryButtonTappedForRowWithIndexPath:`를 써서 `cellForRowAtIndexPath:`로 찾을 수 있는 `UITableViewCell`가 담고 있는 커스텀 악세사리 뷰에 대한 참조를 저장하려고 하는 것이다.

>- **Using associated objects instead of X**, where X is any one the following:
    - [Subclassing](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html) for when inheritance is a more reasonable fit than composition.
    - [Target-Action](https://developer.apple.com/library/ios/documentation/general/conceptual/Devpedia-CocoaApp/TargetAction.html) for adding interaction events to responders.
    - [Gesture Recognizers](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/GestureRecognizer_basics/GestureRecognizer_basics.html) for any situations when target-action doesn't suffice.
    - [Delegation](https://developer.apple.com/library/ios/documentation/general/conceptual/DevPedia-CocoaCore/Delegation.html) when behavior can be delegated to another object.
    - [NSNotification & NSNotificationCenter](http://nshipster.com/nsnotification-and-nsnotificationcenter/) for communicating events across a system in a loosely-coupled way.

- **X대신에 연관객체 사용하기**, X는 다음 어떠한 것들도 될 수 있다.
	- 합성보다 상속이 더 잘 들어 맞는 경우를 위한 [서브 클래싱](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html).
	- 리스폰더에 인터랙션 이벤트를 추가하기 위한 [타겟-액션](https://developer.apple.com/library/ios/documentation/general/conceptual/Devpedia-CocoaApp/TargetAction.html) .
    - 타켓-액션으로 해결하지 못하는 어떤 상황에서 [Gesture Recognizers](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/GestureRecognizer_basics/GestureRecognizer_basics.html).
    - 어떤 행위가 다른 객체에 위임될 수 있을 때에 [델리게이션](https://developer.apple.com/library/ios/documentation/general/conceptual/DevPedia-CocoaCore/Delegation.html).
    - 약한 관계를 유지하면서 시스템 전체에 이벤트를 전달하기 위한 [NSNotification & NSNotificationCenter](http://nshipster.com/nsnotification-and-nsnotificationcenter/).

* * *

>Associated objects should be seen as a method of last resort, rather than a solution in search of a problem (and really, categories themselves really shouldn't be at the top of the toolchain to begin with).

연관 객체는 어떤 문제를 해결하는 해결책이라기 보다 최후의 수단으로 봐야 할 것이다. ( 진짜 진짜 카테고리 자체는 도구 상자 위에 올라와서는 안될 것이다.)


>Like any clever trick, hack, or workaround, there is a natural tendency for one to actively seek out occasions to use it—especially just after learning about it. Do your best to understand and appreciate when it's the right solution, and save yourself the embarrassment of being scornfully asked "why in the name of $DEITY" you decided to go with _that_ solution.

다른 어떤 똑똑한 속임, 해킹, 회피법들처럼 (특히 배움 직후에는) 사용할 수 있는 곳을 찾아 다니는 자연스러운 경향이 있다. 그것이 정공법인 경우에는 최선을 다해서 이해하고 인지해야하고 "$신 은 과연 있는가?"라는 비웃는 질문을 하면서 당황스러움에서 자신 스스로를 구하려면 그냥 _정공법_ 으로 결정해라.
