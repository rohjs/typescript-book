* [타입 호환](#타입-호환)
* [안전성 (Soundness)](#안정성-soundness)
* [구조적 타이핑 (Structural)](#구조적-타이핑-structural)
* [변형 (Variance)](#변형-variance)
* [함수](#함수)
  * [Return Type](#return-type)
  * [Number of arguments](#number-of-arguments)
  * [Optional and rest parameters](#optional-and-rest-parameters)
  * [Types of arguments](#types-of-arguments)
* [Enums](#enums)
* [Classes](#classes)
* [Generics](#generics)
* [FootNote: Invariance](#footnote-invariance)

## 타입 호환

Type Compatibility (as we discuss here) determines if one thing can be assigned to another. E.g. `string` and `number` are not compatible:

타입 호환(Type Compatibility)은 특정 타입을 다른 타입에 할당이 가능한지 결정합니다. 가령, `string`과 `number`는 서로 호환이 되지 않습니다:

```ts
let str: string = "Hello";
let num: number = 123;

str = num; // ERROR: `number`를 `string`에 할당할 수 없습니다.
num = str; // ERROR: `string`을 `number`에 할당할 수 없습니다.
```

## 안정성 (Soundness)

TypeScript's type system is designed to be convenient and allows for *unsound* behaviours e.g. anything can be assigned to `any` which means telling the compiler to allow you to do whatever you want:

TypeScript의 타입 시스템은 편리하게 사용할 수 있도록 고안되어, 종종 *불안전한* 타입 호환을 허용하기도 합니다. 대표적인 예로 `any`가 있습니다. 컴파일러에게 (작성자가) 마음대로 타입을 사용할 것이란 뜻을 전달하는 `any`의 경우 모든 타입에 할당할 수 있기 때문입니다.

```ts
let foo: any = 123;
foo = "Hello";

// 이후
foo.toPrecision(3); // 타입이 `any`이기 때문에 이 경우도 허용됩니다.
```

## 구조적 타이핑 (Structural)

TypeScript objects are structurally typed. This means the *names* don't matter as long as the structures match

TypeScript 객체는 구조적 타이핑을 기반으로 타입 시스템을 갖추고 있습니다. 즉, 구조만 일치한다면 *이름*은 크게 중요하지 않다는 것입니다:

```ts
interface Point {
  x: number,
  y: number
}

class Point2D {
  constructor(public x: number, public y :number) {}
}

let p: Point;
// ㅇㅋ, 구조적 타이핑에 기반하기 때문.
p = new Point2D(1,2);
```

This allows you to create objects on the fly (like you do in vanilla JS) and still have safety whenever it can be inferred.

이를 통해 우리는 TypeScript로 (마치 바닐라 JS에서 그러했듯) 즉시 객체를 생성하고 타입 추론이 가능할 땐 타입 안정성도 유지할 수 있습니다.

Also *more* data is considered fine:

또한 *추가* 데이터도 허용합니다:

```ts
interface Point2D {
  x: number;
  y: number;
}
interface Point3D {
  x: number;
  y: number;
  z: number;
}
var point2D: Point2D = { x: 0, y: 10 }
var point3D: Point3D = { x: 0, y: 10, z: 20 }
function iTakePoint2D(point: Point2D) { /* do something */ }

iTakePoint2D(point2D); // 100% 일치하니 ㅇㅋ
iTakePoint2D(point3D); // 추가적인 정보가 있지만 ㅇㅋ
iTakePoint2D({ x: 0 }); // Error: `y`에 대한 데이터가 빠졌습니다.
```

## 변형 (Variance)

Variance is an easy to understand and important concept for type compatibility analysis.

변형(variance)은 쉽게 이해 가능하고 타입 호환성 분석에 있어 아주 중요한 개념입니다.

For simple types `Base` and `Child`, if `Child` is a child of `Base`, then instances of `Child` can be assigned to a variable of type `Base`.

`Base`와 `Child` 같이 간단한 타입의 경우를 예를 들겠습니다. 가령 `Child`가 `Base`의 자식이라면, `Child`의 인스턴스는 모두 `Base` 타입의 변수에 할당 가능합니다.

> 이는 다형성 101(polymorphism 101)입니다.

In type compatibility of complex types composed of such `Base` and `Child` types depends on where the `Base` and `Child` in similar scenarios is driven by *variance*.

`Base`와 `Child` 등 여러가지 타입으로 구성된 복잡한 타입의 타입 호환성은 유사한 시나리오에서 `Base`와 `Child`가 *변형*에 의해 주도 되는지 위치에 따라 다릅니다.

* Covariant : (co aka joint) only in *same direction*
* Contravariant : (contra aka negative) only in *opposite direction*
* Bivariant : (bi aka both) both co and contra.
* Invariant : if the types aren't exactly the same then they are incompatible.


* 공변共變(Covariant): (co = 접합점) *같은 방향*일 때만
* 반변反變(Contravariant): (contra = 반대의) *반대 방향*일 때만
* 이변二變(Bivariant): (bi = 둘 다) 공변共變, 반변反變을 모두 포함
* 불변不變(Invariant): 타입이 정확하게 일치하지 않으면 무조건 호환 불가

> Note: For a completely sound type system in the presence of mutable data like JavaScript, `invariant` is the only valid option. But as mentioned *convenience* forces us to make unsound choices.

> Note: JavaScript와 같이 변경 가능한 데이터가 존재하는 언어에서 완전하고 견고한 타입 시스템을 적용하려면, `invariant(불변)`만이 유효한 옵션입니다. 하지만 앞서 말씀드렸듯 *편의성*이 우리로 하여금 안전하지 않은 선택을 하게 만듭니다.

## Functions 함수

There are a few subtle things to consider when comparing two functions.

두 함수를 비교할 때 몇몇 미묘한 점들을 고려해야 합니다.

### Return Type 리턴 타입

`covariant`: The return type must contain at least enough data.

`공변(covariant)`: 리턴 타입은 최소한 필수 데이터는 모두 갖춰야 합니다.

```ts
/** 타입 위계질서 */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Two sample functions */
let iMakePoint2D = (): Point2D => ({ x: 0, y: 0 });
let iMakePoint3D = (): Point3D => ({ x: 0, y: 0, z: 0 });

/** Assignment */
iMakePoint2D = iMakePoint3D; // ㅇㅋ
iMakePoint3D = iMakePoint2D; // ERROR: Point2D는 Point3D에 할당할 수 없습니다.
```

### Number of arguments 인자의 갯수

<!-- 더주면 이새끼는 어떻게 처리하라고, 실행맥락과 타입 -->

Fewer arguments are okay (i.e. functions can choose to ignore additional parameters). After all you are guaranteed to be called with at least enough arguments.

인자 수가 적은 건 괜찮습니다. (가령, 함수에서 추가적인 매개 변수를 무시할 수 있으니까요.) 결국 최소한의 필요 인수로만으로도 함수를 호출할 수 있다는 것입니다.

```ts
let iTakeSomethingAndPassItAnErr
  = (x: (err: Error, data: any) => void) => { /* do something */ };

iTakeSomethingAndPassItAnErr(() => null) // ㅇㅋ
iTakeSomethingAndPassItAnErr((err) => null) // ㅇㅋ
iTakeSomethingAndPassItAnErr((err, data) => null) // ㅇㅋ

// ERROR: Argument of type '(err: any, data: any, more: any) => null' is not assignable to parameter of type '(err: Error, data: any) => void'.
// ERROR: 인자로 전달된 '(err: any, data: any, more: any) => null' 타입은 매개 변수 타입 '(err: Error, data: any) => void'에 할당할 수 없습니다.
iTakeSomethingAndPassItAnErr((err, data, more) => null);
```

### Optional and Rest Parameters 선택적 & Rest 파라미터

Optional (pre determined count) and Rest parameters (any count of arguments) are compatible, again for convenience.

선택적 인수(미리 결정된 개수)와 Rest 파라미터(인수가 몇 개가 되든 상관없습니다)은 편의를 위해 호환 가능합니다.

```ts
let foo = (x:number, y: number) => { /* do something */ }
let bar = (x?:number, y?: number) => { /* do something */ }
let bas = (...args: number[]) => { /* do something */ }

foo = bar = bas;
bas = bar = foo;
```

> Note: optional (in our example `bar`) and non optional (in our example `foo`) are only compatible if strictNullChecks is false.

> Note: 선택적 인수(위의 경우 `bar`)와 필수 인수(위의 경우 `foo`)는 `strictNullChecks`가 `false`로 설정된 경우에만 호환 가능합니다.

### Types of arguments 인자 타입

`bivariant` : This is designed to support common event handling scenarios

`이변형(Bivariant)`: 이벤트 핸들링을 지원하기 위해 고안되었습니다.

```ts
/** Event Hierarchy */
/** Event 위계 */
interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

/** Sample event listener */
enum EventType { Mouse, Keyboard }
function addEventListener(eventType: EventType, handler: (n: Event) => void) {
  /* ... */
}

// Unsound, but useful and common. Works as function argument comparison is bivariant
// 불안정하긴 하지만 유용하고 흔히 사용되는 케이스입니다. 함수 인수 비교가 이변성으로 작동합니다.
addEventListener(EventType.Mouse, (e: MouseEvent) => console.log(e.x + "," + e.y));

// Undesirable alternatives in presence of soundness
// 견고함이란 측면에서 바람직하지 않은 대안들

addEventListener(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + "," + (<MouseEvent>e).y));
addEventListener(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + "," + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
// 허용되지 않습니다(명백한 에러입니다). 완전히 호환되지 않는 타입들에게는 타입 안정성이 강제됩니다.

addEventListener(EventType.Mouse, (e: number) => console.log(e));
```

Also makes `Array<Child>` assignable to `Array<Base>` (covariance) as the functions are compatible. Array covariance requires all `Array<Child>` functions to be assignable to `Array<Base>` e.g. `push(t:Child)` is assignable to `push(t:Base)` which is made possible by function argument bivariance.

또한 이는 `Array<Child>`가 `Array<Base>` (공존)에 할당 가능하게 합니다. 왜냐하면 함수가 호환이 되기 때문입니다. 배열 공존성(Array covariance)는 모든 `Array<Child>` 함수들이 `Array<Base>`에 할당이 가능하길 요구하기 때문입니다. 가령 `push(t: Child)`는 `push(t: Base)`에 할당이 가능하고 이는 함수의 인자 이변형(bivariance)이 가능하게 합니다.

**This can be confusing for people coming from other languages** who would expect the following to error but will not in TypeScript:

아래 코드가 에러를 발생시킬 것이라 예상한 **다른 언어에서 넘어온 분**들에게는 헷갈릴 수 있습니다:

```ts
/** Type Hierarchy */
interface Point2D { x: number; y: number; }
interface Point3D { x: number; y: number; z: number; }

/** Two sample functions */
let iTakePoint2D = (point: Point2D) => { /* do something */ }
let iTakePoint3D = (point: Point3D) => { /* do something */ }

iTakePoint3D = iTakePoint2D; // ㅇㅋ : Reasonable 그럴 수 있찌
iTakePoint2D = iTakePoint3D; // ㅇㅋ : WHAT 응?
```

## Enums

* Enums are compatible with numbers, and numbers are compatible with enums.
* Enum은 `number`와 상호 호환이 가능합니다.

```ts
enum Status { Ready, Waiting };

let status = Status.Ready;
let num = 0;

status = num; // ㅇㅋ
num = status; // ㅇㅋ
```

* Enum values from different enum types are considered incompatible. This makes enums useable *nominally* (as opposed to structurally)

* 그러나 다른 `enum` 타입에서의 `enum` 값은 호환되지 않습니다. 그래서 `enum`은 (구조적으로가 아니라) *명목적(이름을 기준으로)*으로 사용할 수 있습니다.

```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
let color = Color.Red;

status = color; // ERROR
```

## Classes

* Only instance members and methods are compared. *constructors* and *statics* play no part.

* 인스턴스 멤버와 메소드만 비교가 가능합니다. *생성자(constructor)*와 *정적 변수(statics)*는 열외입니다.

```ts
class Animal {
  feet: number;
  constructor(name: string, numFeet: number) { /** do something */ }
}

class Size {
  feet: number;
  constructor(meters: number) { /** do something */ }
}

let a: Animal;
let s: Size;

a = s;  // ㅇㅋ
s = a;  // ㅇㅋ
```

* `private` and `protected` members *must originate from the same class*. Such members essentially make the class *nominal*.

* `private`, `protected` 멤버는 *반드시 동일한 `Class`에서 생성되야 합니다.*  그래서 `private`, `protected` 멤버가 `Class`를 *명목적*이게 만듭니다

```ts
/** A class hierarchy */
/** Class 위계 */
class Animal { protected feet: number; }
class Cat extends Animal { }

let animal: Animal;
let cat: Cat;

animal = cat; // ㅇㅋ
cat = animal; // ㅇㅋ

/** Looks just like Animal */
/** Animal이랑 완전 동일하게 생긴 Size class */
class Size { protected feet: number; }

let size: Size;

animal = size; // ERROR
size = animal; // ERROR
```

## Generics

Since TypeScript has a structural type system, type parameters only affect compatibility when used by a member. For example, in the  following `T` has no impact on compatibility:

TypeScript는 구조적 타입 시스템을 갖고 있어, 타입 매개변수는 멤버에 의해 사용될 때만 호환성에 영향을 끼칩니다. 가령, 다음 예시의 `T`는 호환성에 아무런 영향이 없습니다:

```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // okay, y matches structure of x
// ㅇㅋ, y는 x의 구조와 일치하군.
```

However, if `T` is used, it will play a role in compatibility based on its *instantiation* as shown below:

그러나, 만약 `T`가 사용이 된다면, `T`는 아래에서 같이 *인스턴스화*를 기반으로 호환성에 영향을 끼치게 됩니다.

```ts
interface NotEmpty<T> {
  data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // error, x and y are not compatible
// Error: x와 y는 호환되지 않습니다.
```

In cases where generic arguments haven't been *instantiated* they are substituted by `any` before checking compatibility:

제네릭 인자가 *인스턴스화 되지 않은 경우*에 타입 호환성을 체크하기 전에 `any`로 대체됩니다:

```ts
let identity = function<T>(x: T): T {
  // ...
}

let reverse = function<U>(y: U): U {
  // ...
}

identity = reverse;  // Okay because (x: any)=>any matches (y: any)=>any
// ㅇㅋ, (x: any) => any와 (y: any) => any는 일치합니다.
```

Generics involving classes are matched by relevant class compatibility as mentioned before. e.g.

`Class`를 포함하는 Generic은 앞서 언급했듯 관련있는 클래스 끼리의 호환성에 의해 일치됩니다. 가령:

```ts
class List<T> {
  add(val: T) { }
}

class Animal { name: string; }
class Cat extends Animal { meow() { } }

const animals = new List<Animal>();
animals.add(new Animal()); // ㅇㅋ
animals.add(new Cat()); // ㅇㅋ

const cats = new List<Cat>();
cats.add(new Animal()); // Error
cats.add(new Cat()); // ㅇㅋ
```

## FootNote: Invariance 각주: 불변성

We said invariance is the only sound option. Here is an example where both `contra` and `co` variance are shown to be unsafe for arrays.

앞서 견고한 옵션은`불변(invariance)` 밖에 없다고 이야기했습니다. 아래의 예제는 `이변형`과 `공변성`이 배열에 있어 왜 안전하지 않은지를 보여줍니다:

```ts
/** 위계 */
class Animal { constructor(public name: string){} }
class Cat extends Animal { meow() { } }

/** 각 class의 아이템 */
var animal = new Animal("animal");
var cat = new Cat("cat");

/**
 * Demo : 다변형 101
 * Animal <= Cat
 */
animal = cat; // ㅇㅋ
cat = animal; // ERROR: cat extends animal
// ERROR: cat은 animal을 확장합니다.

/** Array of each to demonstrate variance */
/** 분산을 보여주기 위한 각각의 배열 */
let animalArr: Animal[] = [animal];
let catArr: Cat[] = [cat];

/**
 * Obviously Bad : Contravariance 분명히 나쁨: 반변성
 * Animal <= Cat
 * Animal[] >= Cat[]
 */
catArr = animalArr; // Okay if contravariant 반변성일 경우 허용됩니다
catArr[0].meow(); // Allowed but BANG 🔫 at runtime 허용은 되었지만 런타임에서 BANG 🔫 에러가 납니다.


/**
 * Also Bad : covariance 또다른 나쁜 사례: 공변성
 * Animal <= Cat
 * Animal[] <= Cat[]
 */
animalArr = catArr; // Okay if covariant 공변한다면 허용됩니다.
animalArr.push(new Animal('another animal')); // Just pushed an animal into catArr! catArr에 animal을 푸시했습니다!
catArr.forEach(c => c.meow()); // Allowed but BANG 🔫 at runtime 허용은 되었지만 런타임에서 BANG 🔫 에러가 납니다.
```
