---
name: flutter-dashboard-patterns
description: >-
  Production dashboard UI patterns for Flutter: animation tokens system with spring physics and page
  transitions, glassmorphism cards with staggered entrance, gradient stat cards with animated counters,
  shimmer loading skeletons, error state with shake animation, empty state with animated entrance,
  consistent section headers, status badges, store selector with bottom sheet, multi-tenant store
  selection with persistence, schema-based feature visibility, state provider integration with Riverpod,
  and responsive metric card layouts.
license: MIT
metadata:
  author: opencode-agent-kit
  version: '1.0.0'
  target_agent: flutter-developer
  stack:
    - Dart 3
    - Flutter SDK
    - Material Design 3
    - Riverpod
    - GoRouter
    - intl
---

# Flutter Dashboard Patterns

**Target Agent:** @flutter-developer  
**Stack:** Dart 3 · Flutter SDK · Material Design 3 · Riverpod · GoRouter · intl

Production patterns for building data-rich dashboard UIs: animation tokens, glass cards, gradient stat cards, shimmer loading, error/empty states, and multi-tenant store selection.

---

## Table of Contents

1. [Animation Tokens System](#1-animation-tokens-system)
2. [GoRouter Page Transitions](#2-gorouter-page-transitions)
3. [Glassmorphism Card Components](#3-glassmorphism-card-components)
4. [Gradient Stat Card & Animated Counter](#4-gradient-stat-card--animated-counter)
5. [Shimmer Loading & Skeleton](#5-shimmer-loading--skeleton)
6. [Error State with Shake Animation](#6-error-state-with-shake-animation)
7. [Empty State with Staggered Entrance](#7-empty-state-with-staggered-entrance)
8. [Supporting UI Components](#8-supporting-ui-components)
9. [Multi-Tenant Store Selection](#9-multi-tenant-store-selection)
10. [Schema-Based Feature Visibility](#10-schema-based-feature-visibility)
11. [Dashboard Data Flow with Riverpod](#11-dashboard-data-flow-with-riverpod)

---

## 1. Animation Tokens System

Define animation constants in one place so every widget shares a consistent rhythm.

```dart
import 'dart:ui' as ui;
import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class AnimConstants {
  AnimConstants._();

  // ── Durations ──
  static const Duration kInstant = Duration(milliseconds: 100);
  static const Duration kShort = Duration(milliseconds: 200);
  static const Duration kMedium = Duration(milliseconds: 350);
  static const Duration kLong = Duration(milliseconds: 500);
  static const Duration kExtraLong = Duration(milliseconds: 800);

  // ── Curves ──
  static const Curve kDefaultCurve = Curves.easeOutCubic;
  static const Curve kDialogCurve = Curves.easeOutBack;
  static const Curve kSpringCurve = Curves.elasticOut;
  static const Curve kSmoothCurve = Curves.easeInOutCubic;
  static const Curve kEmphasizedCurve = Curves.easeInOutCubic;

  // ── Spring Physics ──
  static const SpringDescription kDefaultSpring = SpringDescription(
    mass: 1, stiffness: 200, damping: 20,
  );
  static const SpringDescription kSnappySpring = SpringDescription(
    mass: 0.8, stiffness: 300, damping: 18,
  );

  // ── Reusable Animations ──

  /// Staggered entrance — items appear one by one with fade+slide.
  static List<Animation<double>> staggeredEntrance(
    AnimationController parent,
    int itemCount, {
    double start = 0.0,
    double interval = 0.12,
    Curve curve = Curves.easeOut,
  }) => List.generate(
    itemCount,
    (i) => CurvedAnimation(
      parent: parent,
      curve: Interval(
        start + i * interval,
        (start + (i + 1) * interval).clamp(0.0, 1.0),
        curve: curve,
      ),
    ),
  );

  /// Slide from below entrance.
  static Animation<Offset> slideFromBelow(
    Animation<double> parent, {
    double distance = 0.15,
  }) => Tween<Offset>(
    begin: Offset(0, distance),
    end: Offset.zero,
  ).animate(parent);
}
```

### Key Animation Decisions

| Token           | Value                     | Why                                       |
| --------------- | ------------------------- | ----------------------------------------- |
| `kShort`        | 200ms                     | Button press, subtle feedback             |
| `kMedium`       | 350ms                     | Page transitions, card entrances          |
| `kLong`         | 500ms                     | Full-screen overlays, dialogs             |
| `kDefaultCurve` | `easeOutCubic`            | Fast start, smooth settle — feels natural |
| `kSnappySpring` | mass: 0.8, stiffness: 300 | Quick tactile feedback on press           |

---

## 2. GoRouter Page Transitions

Custom page transitions for GoRouter that respect `prefers-reduced-motion`.

```dart
class PageTransition {
  PageTransition._();

  /// Subtle fade + scale (for splash/login/dialogs).
  static CustomTransitionPage fadeScalePage({
    required Widget child,
    String? key,
  }) => CustomTransitionPage(
    key: key != null ? ValueKey(key) : null,
    child: child,
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      final reduce =
          ui.PlatformDispatcher.instance.accessibilityFeatures.reduceMotion;
      if (reduce) return child;

      return FadeTransition(
        opacity: animation,
        child: ScaleTransition(
          scale: Tween<double>(begin: 0.95, end: 1).animate(
            CurvedAnimation(
              parent: animation,
              curve: AnimConstants.kDefaultCurve,
            ),
          ),
          child: child,
        ),
      );
    },
    transitionDuration: AnimConstants.kMedium,
  );

  /// Slide from right + fade (drill-down navigation).
  static CustomTransitionPage slidePage({
    required Widget child,
    String? key,
  }) => CustomTransitionPage(
    key: key != null ? ValueKey(key) : null,
    child: child,
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      final reduce =
          ui.PlatformDispatcher.instance.accessibilityFeatures.reduceMotion;
      if (reduce) return child;

      return SlideTransition(
        position: Tween<Offset>(
          begin: const Offset(0.3, 0),
          end: Offset.zero,
        ).animate(CurvedAnimation(
          parent: animation,
          curve: AnimConstants.kSmoothCurve,
        )),
        child: FadeTransition(opacity: animation, child: child),
      );
    },
    transitionDuration: AnimConstants.kMedium,
  );

  /// Fade-only (for full-screen overlays).
  static CustomTransitionPage fadePage({
    required Widget child,
    String? key,
  }) => CustomTransitionPage(
    key: key != null ? ValueKey(key) : null,
    child: child,
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      final reduce =
          ui.PlatformDispatcher.instance.accessibilityFeatures.reduceMotion;
      if (reduce) return child;
      return FadeTransition(opacity: animation, child: child);
    },
    transitionDuration: AnimConstants.kMedium,
  );
}
```

### Usage in GoRouter

```dart
GoRoute(
  path: '/login',
  pageBuilder: (_, _) => PageTransition.fadeScalePage(child: const LoginScreen()),
),
GoRoute(
  path: '/profile',
  parentNavigatorKey: _rootNavigatorKey,
  pageBuilder: (_, _) => PageTransition.slidePage(child: const ProfileScreen()),
),
```

---

## 3. Glassmorphism Card Components

### GlassCard (stateless, static)

```dart
class GlassCard extends StatelessWidget {
  const GlassCard({
    required this.child,
    super.key,
    this.margin,
    this.padding,
    this.borderRadius = 20,
    this.opacity = 0.85,
    this.tint,
    this.hasGlow = false,
    this.onTap,
    this.width,
  });

  final Widget child;
  final EdgeInsetsGeometry? margin;
  final EdgeInsetsGeometry? padding;
  final double borderRadius;
  final double opacity;
  final Color? tint;
  final bool hasGlow;
  final VoidCallback? onTap;
  final double? width;

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    final isLight = Theme.of(context).brightness == Brightness.light;
    final bgColor = tint ?? (isLight ? Colors.white : const Color(0xFF1C1C1E));

    final decoration = BoxDecoration(
      color: bgColor.withValues(alpha: opacity),
      borderRadius: BorderRadius.circular(borderRadius),
      border: Border.all(
        color: hasGlow
            ? cs.primary.withValues(alpha: 0.3)
            : cs.outlineVariant.withValues(alpha: 0.3),
        width: hasGlow ? 1.0 : 0.5,
      ),
      boxShadow: [
        BoxShadow(
          color: hasGlow
              ? cs.primary.withValues(alpha: 0.12)
              : Colors.black.withValues(alpha: 0.05),
          blurRadius: hasGlow ? 24 : 16,
          offset: const Offset(0, 4),
        ),
        BoxShadow(
          color: Colors.black.withValues(alpha: 0.03),
          blurRadius: 8, offset: const Offset(0, 2),
        ),
        if (hasGlow)
          BoxShadow(
            color: cs.primary.withValues(alpha: 0.08),
            blurRadius: 40, spreadRadius: -4,
          ),
      ],
    );

    final card = Container(
      width: width,
      margin: margin,
      padding: padding ?? const EdgeInsets.all(16),
      decoration: decoration,
      child: child,
    );

    if (onTap != null) return GestureDetector(onTap: onTap, child: card);
    return card;
  }
}
```

### OlposGlassCard (animated entrance)

```dart
class OlposGlassCard extends StatefulWidget {
  const OlposGlassCard({
    required this.child,
    super.key,
    this.margin,
    this.padding,
    this.borderRadius = 20,
    this.hasGlow = false,
    this.onTap,
    this.width,
    this.animationDelay = 0,
  });

  final Widget child;
  final EdgeInsetsGeometry? margin, padding;
  final double borderRadius;
  final bool hasGlow;
  final VoidCallback? onTap;
  final double? width;
  final int animationDelay; // 0-based index for staggered entrance

  @override
  State<OlposGlassCard> createState() => _OlposGlassCardState();
}

class _OlposGlassCardState extends State<OlposGlassCard>
    with SingleTickerProviderStateMixin {
  late final AnimationController _ctrl;
  late final Animation<double> _fadeAnim;
  late final Animation<Offset> _slideAnim;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(vsync: this, duration: AnimConstants.kLong);
    _fadeAnim = CurvedAnimation(
      parent: _ctrl,
      curve: Interval(
        widget.animationDelay * 0.1,
        0.3 + widget.animationDelay * 0.1,
        curve: AnimConstants.kDefaultCurve,
      ),
    );
    _slideAnim = Tween<Offset>(
      begin: const Offset(0, 0.06),
      end: Offset.zero,
    ).animate(_fadeAnim);

    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) _ctrl.forward();
    });
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    // ... same glass decoration as GlassCard above ...
    final card = FadeTransition(
      opacity: _fadeAnim,
      child: SlideTransition(
        position: _slideAnim,
        child: Container(/* decoration same as GlassCard */),
      ),
    );

    if (widget.onTap != null) {
      return _PressScaleWrapper(onTap: widget.onTap!, child: card);
    }
    return card;
  }
}
```

### PressScaleWrapper (tactile feedback)

```dart
class _PressScaleWrapper extends StatefulWidget {
  const _PressScaleWrapper({required this.child, required this.onTap});
  final Widget child;
  final VoidCallback onTap;

  @override
  State<_PressScaleWrapper> createState() => _PressScaleWrapperState();
}

class _PressScaleWrapperState extends State<_PressScaleWrapper>
    with SingleTickerProviderStateMixin {
  late final AnimationController _ctrl;
  late final Animation<double> _anim;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(
      vsync: this,
      duration: AnimConstants.kShort,
      upperBound: 0.03,
    );
    _anim = Tween(begin: 0.0, end: -0.025).animate(
      CurvedAnimation(parent: _ctrl, curve: AnimConstants.kDefaultCurve),
    );
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => GestureDetector(
    onTapDown: (_) => _ctrl.forward(),
    onTapUp: (_) => _ctrl.reverse(),
    onTapCancel: () => _ctrl.reverse(),
    onTap: widget.onTap,
    child: AnimatedBuilder(
      animation: _anim,
      builder: (context, child) =>
          Transform.scale(scale: 1.0 + _anim.value, child: child),
      child: widget.child,
    ),
  );
}
```

---

## 4. Gradient Stat Card & Animated Counter

### AnimatedCounter — number roll-up

```dart
class AnimatedCounter extends StatefulWidget {
  const AnimatedCounter({
    required this.value,
    super.key,
    this.prefix = '',
    this.suffix = '',
    this.style,
    this.decimals = 0,
    this.duration = const Duration(milliseconds: 1200),
    this.curve = Curves.easeOutCubic,
  });

  final double value;
  final String prefix, suffix;
  final TextStyle? style;
  final int decimals;
  final Duration duration;
  final Curve curve;

  @override
  State<AnimatedCounter> createState() => _AnimatedCounterState();
}

class _AnimatedCounterState extends State<AnimatedCounter>
    with SingleTickerProviderStateMixin {
  late AnimationController _ctrl;
  late Animation<double> _anim;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(vsync: this, duration: widget.duration);
    _anim = CurvedAnimation(parent: _ctrl, curve: widget.curve);
    _ctrl.forward();
  }

  @override
  void didUpdateWidget(AnimatedCounter oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.value != widget.value) {
      _ctrl.forward(from: 0);
    }
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) => AnimatedBuilder(
    animation: _anim,
    builder: (context, _) {
      final displayed =
          '${widget.prefix}${(widget.value * _anim.value).toStringAsFixed(widget.decimals)}${widget.suffix}';
      return Text(displayed, style: widget.style);
    },
  );
}
```

### GradientStatCard — full card

```dart
class GradientStatCard extends StatefulWidget {
  const GradientStatCard({
    required this.gradient,
    required this.icon,
    required this.label,
    required this.value,
    super.key,
    this.prefix,
    this.suffix,
    this.onTap,
  });

  final LinearGradient gradient;
  final IconData icon;
  final String label;
  final double value;
  final String? prefix, suffix;
  final VoidCallback? onTap;

  @override
  State<GradientStatCard> createState() => _GradientStatCardState();
}

class _GradientStatCardState extends State<GradientStatCard>
    with SingleTickerProviderStateMixin {
  late final AnimationController _ctrl;
  late final Animation<double> _scaleAnim;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(
      vsync: this, duration: AnimConstants.kShort, upperBound: 0.03,
    );
    _scaleAnim = Tween(begin: 0.0, end: -0.025).animate(
      CurvedAnimation(parent: _ctrl, curve: AnimConstants.kDefaultCurve),
    );
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final card = AnimatedBuilder(
      animation: _scaleAnim,
      builder: (context, _) => Transform.scale(
        scale: 1.0 + _scaleAnim.value,
        child: DecoratedBox(
          decoration: BoxDecoration(
            gradient: widget.gradient,
            borderRadius: BorderRadius.circular(20),
            boxShadow: [
              BoxShadow(
                color: widget.gradient.colors.first.withValues(alpha: 0.3),
                blurRadius: 16, offset: const Offset(0, 6),
              ),
            ],
          ),
          child: Padding(
            padding: const EdgeInsets.all(20),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Container(
                  width: 36, height: 36,
                  decoration: BoxDecoration(
                    color: Colors.white.withValues(alpha: 0.2),
                    borderRadius: BorderRadius.circular(10),
                  ),
                  child: Icon(widget.icon, color: Colors.white, size: 20),
                ),
                const Spacer(),
                AnimatedCounter(
                  value: widget.value,
                  prefix: widget.prefix ?? '',
                  suffix: widget.suffix ?? '',
                  style: const TextStyle(
                    fontSize: 28, fontWeight: FontWeight.w800,
                    color: Colors.white, height: 1.1,
                  ),
                ),
                const SizedBox(height: 4),
                Text(
                  widget.label,
                  style: TextStyle(
                    fontSize: 13, fontWeight: FontWeight.w500,
                    color: Colors.white.withValues(alpha: 0.85),
                  ),
                ),
              ],
            ),
          ),
        ),
      ),
    );

    if (widget.onTap != null) {
      return GestureDetector(
        onTapDown: (_) => _ctrl.forward(),
        onTapUp: (_) => _ctrl.reverse(),
        onTapCancel: () => _ctrl.reverse(),
        onTap: widget.onTap,
        child: card,
      );
    }
    return card;
  }
}
```

### Metric Card Layout Patterns

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│  Hero Metric    │  │  Metric Pair    │  │  Metric Row     │
│                 │  │                 │  │                 │
│  ┌───────────┐  │  │ ┌──────┐┌──────┐│  │ ┌──┐ ┌──┐ ┌──┐ │
│  │  Icon      │  │  │ │Omzet ││Profit││  │ │R │ │P │ │S │ │
│  │  Rp 50,000 │  │  │ │+12%  ││+8%   ││  │ │e │ │r │ │e │ │
│  │  Total     │  │  │ └──────┘└──────┘│  │ │v │ │r │ │r │ │
│  └───────────┘  │  │                 │  │ │  │ │o │ │v │ │
└─────────────────┘  └─────────────────┘  │ └──┘ └──┘ └──┘ │
                                           └─────────────────┘
```

Use `GradientStatCard` for Hero Metrics (single big number), and inline `_statChip` / `_amountChip` widgets for pairs/rows.

---

## 5. Shimmer Loading & Skeleton

### Simple Shimmer Block

```dart
class AppShimmer extends StatelessWidget {
  const AppShimmer({
    super.key,
    this.width = double.infinity,
    this.height = 16,
    this.borderRadius = 8,
  });

  final double width, height, borderRadius;

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    return Container(
      width: width, height: height,
      decoration: BoxDecoration(
        color: cs.surfaceContainerHigh,
        borderRadius: BorderRadius.circular(borderRadius),
      ),
    );
  }
}
```

### Loading State with Animation

```dart
class AppLoading extends StatefulWidget {
  const AppLoading({super.key, this.width, this.height, this.margin});
  final double? width, height;
  final EdgeInsetsGeometry? margin;

  @override
  State<AppLoading> createState() => _AppLoadingState();
}

class _AppLoadingState extends State<AppLoading>
    with SingleTickerProviderStateMixin {
  late final AnimationController _ctrl;
  late final Animation<double> _anim;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(
      vsync: this, duration: const Duration(milliseconds: 1500),
    )..repeat();
    _anim = Tween(begin: 0.0, end: 1.0).animate(_ctrl);
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    return AnimatedBuilder(
      animation: _anim,
      builder: (context, _) => Container(
        width: widget.width ?? double.infinity,
        height: widget.height ?? 120,
        margin: widget.margin ?? const EdgeInsets.symmetric(
          horizontal: 16, vertical: 8,
        ),
        child: Center(
          child: SizedBox(
            width: 24, height: 24,
            child: CircularProgressIndicator(
              strokeWidth: 2.5,
              color: cs.primary.withValues(alpha: 0.6),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## 6. Error State with Shake Animation

```dart
class AppErrorWidget extends StatefulWidget {
  const AppErrorWidget({required this.message, super.key, this.onRetry});
  final String message;
  final VoidCallback? onRetry;

  @override
  State<AppErrorWidget> createState() => _AppErrorWidgetState();
}

class _AppErrorWidgetState extends State<AppErrorWidget>
    with SingleTickerProviderStateMixin {
  late final AnimationController _ctrl;
  late final Animation<double> _shakeAnim;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(
      vsync: this, duration: const Duration(milliseconds: 600),
    );
    _shakeAnim = Tween<double>(begin: 0, end: 4).animate(
      CurvedAnimation(parent: _ctrl, curve: Curves.elasticIn),
    );
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) _ctrl.forward();
    });
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(24),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            AnimatedBuilder(
              animation: _shakeAnim,
              builder: (context, _) => Transform.translate(
                offset: Offset(
                  _shakeAnim.value *
                      (DateTime.now().millisecondsSinceEpoch.isEven ? 1 : -1),
                  0,
                ),
                child: Container(
                  width: 72, height: 72,
                  decoration: BoxDecoration(
                    color: cs.errorContainer.withValues(alpha: 0.5),
                    borderRadius: BorderRadius.circular(24),
                  ),
                  child: Icon(
                    Icons.error_outline_rounded,
                    size: 32, color: cs.error,
                  ),
                ),
              ),
            ),
            const SizedBox(height: 20),
            Text(
              widget.message,
              textAlign: TextAlign.center,
              style: TextStyle(
                fontSize: 15, color: cs.onSurfaceVariant, height: 1.4,
              ),
            ),
            if (widget.onRetry != null) ...[
              const SizedBox(height: 20),
              FilledButton.tonalIcon(
                onPressed: widget.onRetry,
                icon: const Icon(Icons.refresh_rounded, size: 18),
                label: const Text('Try Again'),
                style: FilledButton.styleFrom(
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(14),
                  ),
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

---

## 7. Empty State with Staggered Entrance

```dart
class EmptyStateWidget extends StatefulWidget {
  const EmptyStateWidget({
    required this.message,
    super.key,
    this.icon = Icons.inbox_outlined,
    this.actionLabel,
    this.onAction,
  });

  final IconData icon;
  final String message;
  final String? actionLabel;
  final VoidCallback? onAction;

  @override
  State<EmptyStateWidget> createState() => _EmptyStateWidgetState();
}

class _EmptyStateWidgetState extends State<EmptyStateWidget>
    with SingleTickerProviderStateMixin {
  late final AnimationController _ctrl;
  late final Animation<double> _fadeAnim, _iconAnim;
  late final Animation<Offset> _slideAnim;

  @override
  void initState() {
    super.initState();
    _ctrl = AnimationController(vsync: this, duration: AnimConstants.kLong);
    _iconAnim = CurvedAnimation(
      parent: _ctrl,
      curve: const Interval(0, 0.5, curve: Curves.easeOutBack),
    );
    _fadeAnim = CurvedAnimation(
      parent: _ctrl,
      curve: const Interval(0.3, 0.8, curve: Curves.easeOut),
    );
    _slideAnim = Tween<Offset>(
      begin: const Offset(0, 0.05), end: Offset.zero,
    ).animate(_fadeAnim);

    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) _ctrl.forward();
    });
  }

  @override
  void dispose() {
    _ctrl.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    return Center(
      child: Padding(
        padding: const EdgeInsets.all(32),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            ScaleTransition(
              scale: _iconAnim,
              child: Container(
                width: 80, height: 80,
                decoration: BoxDecoration(
                  color: cs.surfaceContainerHigh,
                  borderRadius: BorderRadius.circular(24),
                ),
                child: Icon(widget.icon, size: 36, color: cs.outlineVariant),
              ),
            ),
            const SizedBox(height: 20),
            FadeTransition(
              opacity: _fadeAnim,
              child: SlideTransition(
                position: _slideAnim,
                child: Text(
                  widget.message,
                  textAlign: TextAlign.center,
                  style: TextStyle(
                    fontSize: 15, color: cs.onSurfaceVariant, height: 1.4,
                  ),
                ),
              ),
            ),
            if (widget.actionLabel != null && widget.onAction != null) ...[
              const SizedBox(height: 20),
              FadeTransition(
                opacity: _fadeAnim,
                child: FilledButton.tonalIcon(
                  onPressed: widget.onAction,
                  icon: const Icon(Icons.add_rounded, size: 18),
                  label: Text(widget.actionLabel!),
                  style: FilledButton.styleFrom(
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(14),
                    ),
                  ),
                ),
              ),
            ],
          ],
        ),
      ),
    );
  }
}
```

---

## 8. Supporting UI Components

### SectionHeader

```dart
class SectionHeader extends StatelessWidget {
  const SectionHeader({
    required this.title,
    super.key,
    this.subtitle,
    this.trailing,
  });

  final String title;
  final String? subtitle;
  final Widget? trailing;

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    return Padding(
      padding: const EdgeInsets.symmetric(horizontal: 4, vertical: 8),
      child: Row(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Expanded(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  title,
                  style: Theme.of(context).textTheme.titleSmall?.copyWith(
                    fontWeight: FontWeight.w600, color: cs.onSurface,
                  ),
                ),
                if (subtitle != null) ...[
                  const SizedBox(height: 2),
                  Text(subtitle!,
                    style: Theme.of(context).textTheme.bodySmall
                        ?.copyWith(color: cs.onSurfaceVariant),
                  ),
                ],
              ],
            ),
          ),
          if (trailing != null) trailing!,
        ],
      ),
    );
  }
}
```

### StatusBadge

```dart
class StatusBadge extends StatelessWidget {
  const StatusBadge({
    required this.label,
    required this.color,
    super.key,
    this.icon,
  });

  final String label;
  final Color color;
  final IconData? icon;

  @override
  Widget build(BuildContext context) => Container(
    padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 4),
    decoration: BoxDecoration(
      color: color.withValues(alpha: 0.12),
      borderRadius: BorderRadius.circular(20),
    ),
    child: Row(
      mainAxisSize: MainAxisSize.min,
      children: [
        if (icon != null) ...[
          Icon(icon, size: 14, color: color),
          const SizedBox(width: 4),
        ],
        Text(
          label.toUpperCase(),
          style: TextStyle(
            fontSize: 10, fontWeight: FontWeight.w700,
            color: color, letterSpacing: 0.5,
          ),
        ),
      ],
    ),
  );
}
```

---

## 9. Multi-Tenant Store Selection

When a dashboard serves multiple stores/locations, the user needs to select which store's data to view. This selection must persist across app restarts.

### Store Selection Notifier (Riverpod)

```dart
const _storeIdKey = 'selected_store_id';

/// Persisted store ID — loaded on demand by ownedStoresProvider.
final persistedStoreIdProvider = FutureProvider<String?>((ref) async {
  try {
    return await const FlutterSecureStorage().read(key: _storeIdKey);
  } catch (_) {
    return null;
  }
});

/// Currently selected store ID — persisted across sessions.
class StoreSelection extends Notifier<String?> {
  @override
  String? build() => null;

  void selectStore(String storeId) {
    state = storeId;
    const FlutterSecureStorage().write(key: _storeIdKey, value: storeId);
  }

  void clear() {
    state = null;
    const FlutterSecureStorage().delete(key: _storeIdKey);
  }
}

final storeSelectionProvider = NotifierProvider<StoreSelection, String?>(
  StoreSelection.new,
);

/// Fetches owned stores — restores persisted selection or auto-selects first.
final ownedStoresProvider = FutureProvider.autoDispose<List<OwnedStore>>((
  ref,
) async {
  final authState = ref.watch(authProvider);
  if (authState is! AuthAuthenticated) return [];

  final persisted = await ref.watch(persistedStoreIdProvider.future);
  final dataSource = OwnedStoreDataSource();
  final json = await dataSource.getOwnedStores(userId);

  var list = /* parse JSON → List<OwnedStore> */ [];

  final notifier = ref.read(storeSelectionProvider.notifier);
  final current = ref.read(storeSelectionProvider);

  if (persisted != null && list.any((s) => s.storeId == persisted)) {
    if (current != persisted) notifier.selectStore(persisted);
  } else if (current == null && list.isNotEmpty) {
    notifier.selectStore(list.first.storeId);
  }

  return list;
});
```

### Store Selector Bottom Sheet

```dart
void _showStoreSelector(BuildContext context, WidgetRef ref) {
  showOlposBottomSheet(
    context: context,
    builder: (_) => StoreSelectorSheet(stores: stores),
  );
}

class StoreSelectorSheet extends ConsumerWidget {
  const StoreSelectorSheet({required this.stores, super.key});
  final List<OwnedStore> stores;

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final selectedId = ref.watch(storeSelectionProvider);
    final cs = Theme.of(context).colorScheme;

    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Padding(
          padding: const EdgeInsets.all(16),
          child: Text('Pilih Toko',
            style: Theme.of(context).textTheme.titleMedium,
          ),
        ),
        ...stores.map((store) => ListTile(
          leading: Icon(
            store.storeId == selectedId
                ? Icons.radio_button_checked
                : Icons.radio_button_unchecked,
            color: store.storeId == selectedId
                ? cs.primary : cs.onSurfaceVariant,
          ),
          title: Text(store.store.name),
          subtitle: store.store.phone != null
              ? Text(store.store.phone!)
              : null,
          onTap: () {
            ref.read(storeSelectionProvider.notifier)
                .selectStore(store.storeId);
            context.pop();
          },
        )),
      ],
    );
  }
}
```

---

## 10. Schema-Based Feature Visibility

When a single codebase serves multiple business types (retail, laundry, restaurant, workshop), use a `Provider` to compute which UI sections are visible based on the store's schema.

```dart
/// Exposes the schema of the currently selected store.
final storeSchemaProvider = Provider<String?>((ref) {
  final stores = ref.watch(ownedStoresProvider).valueOrNull ?? [];
  final selectedId = ref.watch(storeSelectionProvider);
  // ... find store by selectedId, return store.schema
});

/// Dashboard feature visibility based on store schema.
class SalesDashboardVisibility {
  final bool showConsignment;
  final bool showTechnician;
  final bool showServiceIncome;

  const SalesDashboardVisibility({
    required this.showConsignment,
    required this.showTechnician,
    required this.showServiceIncome,
  });

  static const none = SalesDashboardVisibility(
    showConsignment: false,
    showTechnician: false,
    showServiceIncome: false,
  );
}

final salesDashboardVisibilityProvider =
    Provider<SalesDashboardVisibility>((ref) {
  final schema = ref.watch(storeSchemaProvider);

  switch (schema) {
    case 'SIMASKO':
      return const SalesDashboardVisibility(
        showConsignment: true,
        showTechnician: true,
        showServiceIncome: true,
      );
    case 'LAUNDRY':
    case 'BARBERSHOP':
      return const SalesDashboardVisibility(
        showConsignment: false,
        showTechnician: false,
        showServiceIncome: true,
      );
    case 'RETAIL':
    case 'MINIMARKET':
    case 'FASHION':
    case 'APOTEK':
      return const SalesDashboardVisibility(
        showConsignment: true,
        showTechnician: false,
        showServiceIncome: false,
      );
    default:
      return SalesDashboardVisibility.none;
  }
});
```

### Usage in the Dashboard Screen

```dart
final visibility = ref.watch(salesDashboardVisibilityProvider);

if (visibility.showTechnician) {
  TechnicianSection();
}
if (visibility.showConsignment) {
  ConsignmentSection();
}
```

### Derived Schema Providers

```dart
final isSchemaLaundryProvider = Provider<bool>((ref) {
  final schema = ref.watch(storeSchemaProvider);
  return schema == 'LAUNDRY';
});

final isVariantProvider = Provider<bool>((ref) {
  final stores = ref.watch(ownedStoresProvider).valueOrNull ?? [];
  final selectedId = ref.watch(storeSelectionProvider);
  // ... find store and return store.store.isVariant
});
```

---

## 11. Dashboard Data Flow with Riverpod

### Complete Dashboard Screen Pattern

```dart
class SalesDashboardScreen extends ConsumerWidget {
  const SalesDashboardScreen({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final visibility = ref.watch(salesDashboardVisibilityProvider);
    final summaryAsync = ref.watch(salesSummaryProvider);

    return Scaffold(
      body: RefreshIndicator(
        onRefresh: () => ref.refresh(salesSummaryProvider.future),
        child: ListView(
          padding: const EdgeInsets.all(16),
          children: [
            // Hero metric cards (always visible)
            SectionHeader(title: l10n.overview),
            Row(
              children: [
                Expanded(
                  child: GradientStatCard(
                    gradient: OlposGradients.brand,
                    icon: Icons.trending_up_rounded,
                    label: l10n.totalRevenue,
                    value: summaryAsync.valueOrNull?.totalRevenue ?? 0,
                    prefix: 'Rp ',
                  ),
                ),
                const SizedBox(width: 12),
                Expanded(
                  child: GradientStatCard(
                    gradient: OlposGradients.success,
                    icon: Icons.account_balance_wallet_rounded,
                    label: l10n.totalProfit,
                    value: summaryAsync.valueOrNull?.totalProfit ?? 0,
                    prefix: 'Rp ',
                  ),
                ),
              ],
            ),
            const SizedBox(height: 16),

            // Conditional sections
            if (visibility.showTechnician)
              TechnicianSection(),
            if (visibility.showConsignment)
              ConsignmentSection(),
            if (visibility.showServiceIncome)
              ServiceIncomeSection(),

            // Error/loading/empty handled via AsyncValue.when when needed
          ],
        ),
      ),
    );
  }
}
```

### Cross-Provider Invalidation on Store Change

```dart
// When the user switches stores, invalidate all data providers:
ref.listen(storeSelectionProvider, (prev, next) {
  if (prev != next) {
    ref.invalidate(salesSummaryProvider);
    ref.invalidate(financeHistoryProvider);
    ref.invalidate(hrDataProvider);
  }
});
```

### Cross-App Auth Change: Clear Store + Invalidate All

```dart
void logout() {
  ref.read(storeSelectionProvider.notifier).clear();
  clearBusinessAnalysisCache();
  state = const AuthInitial();
}
```

---

## Complete pubspec Dependencies

```yaml
dependencies:
  flutter_riverpod: ^2.6.1
  go_router: ^14.8.0
  flutter_secure_storage: ^9.2.4
  intl: ^0.20.2
  fl_chart: ^0.70.2 # for revenue charts
  shimmer: ^3.0.0 # enhanced loading skeletons

dev_dependencies:
  flutter_test:
    sdk: flutter
  mocktail: ^1.0.4
```

---

## Design Decisions Reference

| Component           | Choice                                     | Alternative Considered | Why                                                                             |
| ------------------- | ------------------------------------------ | ---------------------- | ------------------------------------------------------------------------------- |
| Card style          | Glassmorphism                              | Solid cards            | Brand-consistent, modern feel. Glass is intentional (not default).              |
| Card entrance       | Slide + fade staggered                     | Instant render         | Each card entering tells the user data is fresh/live.                           |
| Number animation    | `AnimatedBuilder` with lerp                | `AnimatedSwitcher`     | Smooth continuous animation vs. abrupt tween.                                   |
| Loading placeholder | Pulse opacity                              | Shimmer package        | Lighter dependency, simpler code. Use `shimmer` for richer effects when needed. |
| Error shake         | `elasticIn` curve                          | Static error icon      | Draws attention without being annoying.                                         |
| Store selection     | Riverpod Notifier + `FlutterSecureStorage` | SharedPreferences      | Secure token storage, no plaintext IDs.                                         |
| Feature visibility  | Riverpod `Provider`                        | If/else in widget      | Testable, reactive, debuggable in isolation.                                    |
