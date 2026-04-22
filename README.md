# Part 1: Tap Gestures in SwiftUI
### iOS Labs Contribution Series SwiftUI Interactions

> **Series:** SwiftUI Interactions from Zero to Real-World  
> **Part:** 1 of 3  
> **iOS Target:** iOS 17+ 

---

## 🌍 Why Tap Gestures? (Real World Context)

Think about the last time you used your phone.

You double tapped a photo to like it on Instagram. You long pressed a message to react. You tapped a card to expand it. You swiped to dismiss a notification.

**Every single one of those is a gesture.**

In SwiftUI, gestures are your primary tool for making apps feel *alive and responsive*. They turn static UI into something users can interact with naturally — because they mirror how people interact with the physical world.

Before we build a TikTok-style feed (Part 2) or add animations (Part 3), we need to understand the foundation: **how SwiftUI handles gestures, and how to use them confidently.**

---

## 📦 What's Covered in This Part

| # | Gesture | Real-World Analogy |
|---|---------|-------------------|
| 1 | `TapGesture` | Pressing a doorbell |
| 2 | `LongPressGesture` | Holding a button in an elevator |
| 3 | `DragGesture` | Sliding a physical switch |
| 4 | `MagnifyGesture` | Pinching a printed photo to "zoom" |
| 5 | `RotationGesture` | Rotating a dial |
| 6 | Combining Gestures | Using two hands to interact |

---

## 🧱 The Basics — How Gestures Work in SwiftUI

SwiftUI attaches gestures to views using the `.gesture()` modifier. Think of it like putting a sticker sensor on top of a view — whenever the user interacts with that area, the sensor fires.

```swift
Text("Tap me!")
    .gesture(
        TapGesture()
            .onEnded {
                print("Tapped!")
            }
    )
```

There's also a shorthand for the most common ones:

```swift
Text("Tap me!")
    .onTapGesture {
        print("Tapped!")
    }
```

Both do the same thing. The `.onTapGesture` shorthand is cleaner for simple cases.

### 🔑 Key Concept: State Drives UI

In SwiftUI, gestures almost always update `@State`. The state change then triggers a UI update. This is the core mental model **never try to directly manipulate the UI. Change state, and let SwiftUI re-render.**

```swift
@State private var isLiked = false

Image(systemName: isLiked ? "heart.fill" : "heart")
    .foregroundStyle(isLiked ? .red : .gray)
    .onTapGesture {
        isLiked.toggle()
    }
```

---

## 🛠️ Step-by-Step: All Gesture Types

---

### 1. `TapGesture` Single & Double Tap

The most common gesture. You can detect single taps or configure it for double taps.

**Single Tap — Instagram-style Like Button**

```swift
import SwiftUI

struct LikeButtonView: View {
    @State private var isLiked = false

    var body: some View {
        Image(systemName: isLiked ? "heart.fill" : "heart")
            .font(.system(size: 40))
            .foregroundStyle(isLiked ? .red : .gray)
            .onTapGesture {
                isLiked.toggle()
            }
    }
}
```

**Double Tap Tap twice to zoom (photo viewer style)**

```swift
struct DoubleTapView: View {
    @State private var isZoomed = false

    var body: some View {
        Image("samplePhoto")
            .resizable()
            .scaledToFit()
            .scaleEffect(isZoomed ? 2.0 : 1.0)
            .animation(.spring(duration: 0.3), value: isZoomed)
            .onTapGesture(count: 2) {         // 👈 count: 2 = double tap
                isZoomed.toggle()
            }
    }
}
```

> 💡 **Tip:** When combining single and double tap on the same view, always attach double tap *first* SwiftUI reads gestures top to bottom and the single tap will fire before the double tap gets a chance otherwise.

```swift
.onTapGesture(count: 2) { /* double tap */ }
.onTapGesture(count: 1) { /* single tap */ }
```

---

### 2. `LongPressGesture` Hold to Trigger

Long press is perfect for context menus, force action confirmations, or "hold to record" mechanics.

**Basic Long Press**

```swift
struct LongPressView: View {
    @State private var isPressed = false

    var body: some View {
        Circle()
            .fill(isPressed ? Color.orange : Color.blue)
            .frame(width: 100, height: 100)
            .scaleEffect(isPressed ? 0.9 : 1.0)
            .animation(.easeInOut(duration: 0.2), value: isPressed)
            .onLongPressGesture(minimumDuration: 0.5) {
                isPressed.toggle()
            }
    }
}
```

**Long Press with In Progress Feedback**

Show the user *while* they're pressing not just after:

```swift
struct RecordButtonView: View {
    @State private var isRecording = false

    var body: some View {
        Circle()
            .fill(isRecording ? Color.red : Color.gray)
            .frame(width: 80, height: 80)
            .overlay(
                Text(isRecording ? "REC" : "Hold")
                    .foregroundStyle(.white)
                    .font(.caption.bold())
            )
            .onLongPressGesture(
                minimumDuration: 1.0,
                perform: {
                    isRecording = false          // long press completed
                },
                onPressingChanged: { pressing in
                    isRecording = pressing       // fires as user presses/releases
                }
            )
    }
}
```

---

### 3. `DragGesture` Swipe & Drag

Drag gestures give you real time position data as the user moves their finger.

**Draggable Card**

```swift
struct DraggableCardView: View {
    @State private var offset: CGSize = .zero

    var body: some View {
        RoundedRectangle(cornerRadius: 20)
            .fill(Color.indigo)
            .frame(width: 200, height: 120)
            .offset(offset)                             // 👈 apply drag offset
            .gesture(
                DragGesture()
                    .onChanged { value in
                        offset = value.translation      // follow finger
                    }
                    .onEnded { _ in
                        withAnimation(.spring) {
                            offset = .zero              // snap back to origin
                        }
                    }
            )
    }
}
```

**Key `DragGesture` values you get:**

| Property | What it gives you |
|---|---|
| `value.translation` | How far from start (x, y) |
| `value.location` | Current finger position on screen |
| `value.startLocation` | Where the drag began |
| `value.velocity` | Speed of the drag (iOS 17+) |

> 💡 **iOS 17 Bonus:** `value.velocity` lets you detect a flick vs. a slow drag - great for swipe-to-dismiss animations that feel natural.

---

### 4. `MagnifyGesture` Pinch to Zoom

Used in photo viewers, maps, and any content you want users to zoom into.

```swift
struct PinchZoomView: View {
    @State private var scale: CGFloat = 1.0

    var body: some View {
        Image("samplePhoto")
            .resizable()
            .scaledToFit()
            .scaleEffect(scale)
            .gesture(
                MagnifyGesture()                        // called MagnificationGesture pre-iOS 17
                    .onChanged { value in
                        scale = value.magnification     // value is a CGFloat multiplier
                    }
                    .onEnded { _ in
                        withAnimation(.spring) {
                            scale = max(1.0, scale)     // prevent zoom below original size
                        }
                    }
            )
    }
}
```

> ⚠️ **Note:** In iOS 17+, `MagnificationGesture` was renamed to `MagnifyGesture`. Both work for now, but prefer `MagnifyGesture` going forward.

---

### 5. `RotationGesture` Twist & Rotate

Rotate any view using two fingers great for sticker editors, image tools, or creative apps.

```swift
struct RotatableEmojiView: View {
    @State private var rotation: Angle = .zero

    var body: some View {
        Text("🌍")
            .font(.system(size: 80))
            .rotationEffect(rotation)
            .gesture(
                RotateGesture()                         // called RotationGesture pre-iOS 17
                    .onChanged { value in
                        rotation = value.rotation
                    }
            )
    }
}
```

> ⚠️ **Note:** Like `MagnifyGesture`, iOS 17 renamed `RotationGesture` → `RotateGesture`.

---

### 6. Combining Gestures Two Gestures at Once

SwiftUI lets you compose gestures in three ways:

| Composition | Modifier | Behavior |
|---|---|---|
| **Simultaneously** | `.simultaneously(with:)` | Both gestures active at the same time |
| **Sequentially** | `.sequenced(before:)` | Second gesture starts only after first completes |
| **Exclusively** | `.exclusively(before:)` | Only one fires; first match wins |

**Pinch + Rotate Simultaneously (like Photos app)**

```swift
struct PinchAndRotateView: View {
    @State private var scale: CGFloat = 1.0
    @State private var rotation: Angle = .zero

    var body: some View {
        Image("samplePhoto")
            .resizable()
            .scaledToFit()
            .scaleEffect(scale)
            .rotationEffect(rotation)
            .gesture(
                MagnifyGesture()
                    .onChanged { value in scale = value.magnification }
                    .simultaneously(with:
                        RotateGesture()
                            .onChanged { value in rotation = value.rotation }
                    )
            )
    }
}
```

---

## 🧠 Advanced Tips & Best Practices

### ✅ Use `@Observable` for Gesture State (iOS 17+)

If gesture state is shared across multiple views, move it to an `@Observable` class instead of keeping it in `@State`:

```swift
@Observable
class GestureViewModel {
    var isLiked = false
    var dragOffset: CGSize = .zero

    func toggleLike() {
        isLiked.toggle()
    }
}

struct ContentView: View {
    var viewModel = GestureViewModel()

    var body: some View {
        LikeButtonView(viewModel: viewModel)
    }
}
```

### ✅ Prioritize Gestures When Views Overlap

If parent and child views both have gestures, use `.highPriorityGesture()` to control which wins:

```swift
ParentView()
    .gesture(TapGesture().onEnded { print("Parent tapped") })
    .overlay(
        ChildView()
            .highPriorityGesture(
                TapGesture().onEnded { print("Child tapped — I win!") }
            )
    )
```

### ✅ Add Feedback with `.sensoryFeedback` (iOS 17+)

Make gestures feel physical with haptics one line of code:

```swift
@State private var isLiked = false

Image(systemName: isLiked ? "heart.fill" : "heart")
    .onTapGesture { isLiked.toggle() }
    .sensoryFeedback(.impact(weight: .medium), trigger: isLiked)  // 👈 haptic on change
```

### ✅ Don't Block Scroll Views

Drag gestures inside a `ScrollView` can conflict. Use `.simultaneousGesture()` instead of `.gesture()` so both can coexist:

```swift
ScrollView {
    CardView()
        .simultaneousGesture(
            DragGesture().onChanged { ... }
        )
}
```

---

## 🎯 Quick Reference Gesture Cheat Sheet

```
TapGesture()                    → Single tap
TapGesture(count: 2)            → Double tap
LongPressGesture(min: 0.5)      → Hold for 0.5s
DragGesture()                   → Drag / swipe (use .translation, .velocity)
MagnifyGesture()                → Pinch to zoom
RotateGesture()                 → Two-finger rotation

Combining:
  .simultaneously(with:)        → Both at once
  .sequenced(before:)           → One then the other
  .exclusively(before:)         → First match wins

Priority:
  .gesture()                    → Normal priority
  .highPriorityGesture()        → Beats child gestures
  .simultaneousGesture()        → Plays nicely with ScrollView
```

---

## 🤝 Contributing

Found a bug? Have a better example? Want to add a gesture type we missed?

1. Fork this repo
2. Create a branch: `git checkout -b improve/tap-gestures`
3. Make your changes
4. Open a PR with a clear description

All skill levels welcome this is a learner first repo. 🙌

---

## 👋 About the Author

Hey, I'm **Preet Pambhar**  an iOS developer who believes the best way to grow is to learn in public, share what you know, and build alongside a community.

This series is part of my contribution to **iOS Labs**, where the goal is simple: make SwiftUI concepts approachable, practical, and genuinely useful whether you're just starting out or already shipping apps.

If this helped you understand gestures even a little better, that's a win. And if you have feedback, improvements, or just want to connect I'd love to hear from you.

Let's keep building. 🚀

**Preet Pambhar**
[![GitHub](https://img.shields.io/badge/GitHub-preetpambhar-181717?style=flat&logo=github)](https://github.com/preetpambhar)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-preet--pambhar-0A66C2?style=flat&logo=linkedin)](https://www.linkedin.com/in/preet-pambhar)

---

*Made with ❤️ for the iOS Labs contribution series.*
