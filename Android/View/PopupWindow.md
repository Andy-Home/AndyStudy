# PopupWindow (Android 26)

### 源码解析

&emsp;&emsp;当使用 *new PopupWindow(View contentView)* 等类似方法时，最终会调用同一个构造函数
```java
public PopupWindow(View contentView, int width, int height, boolean focusable) {
        if (contentView != null) {
            mContext = contentView.getContext();
            mWindowManager = (WindowManager) mContext
                              .getSystemService(Context.WINDOW_SERVICE);
        }

        setContentView(contentView);
        setWidth(width);
        setHeight(height);
        setFocusable(focusable);
    }
```
&emsp;&emsp;其中需要注意的时setContentView方法，阅读源码后发现，它在LOLLIPOP_MR1之前
的版本与后续的版本是有区别的。
区别代码如下:
```java
public void setContentView(View contentView) {
        ......
        // Setting the default for attachedInDecor based on SDK version here
        // instead of in the constructor since we might not have the context
        // object in the constructor. We only want to set default here if the
        // app hasn't already set the attachedInDecor.
        if (mContext != null && !mAttachedInDecorSet) {
            // Attach popup window in decor frame of parent window by default for
            // {@link Build.VERSION_CODES.LOLLIPOP_MR1} or greater. Keep current
            // behavior of not attaching to decor frame for older SDKs.
            setAttachedInDecor(mContext.getApplicationInfo().targetSdkVersion
                    >= Build.VERSION_CODES.LOLLIPOP_MR1);
        }

    }
```
&emsp;&emsp;当版本大于或者等于LOLLIPOP_MR1版本时，PopupWindow会依附于父Window的
Decor frame中。而小于LOLLIPOP_MR1版本的依然保持现状，不依附于Decor frame。

&emsp;&emsp;此时，PopupWindow类已经初始化成功，但是对应的PopupWindow View并没有创建
成功，PopupWindow的最终呈现的View与它的两个内部类有关 *PopupBackgroundView* 和
*PopupDecorView*，它们是什么时候创建的呢？

&emsp;&emsp;跟着平常使用PopupWindow的流程，此时我们需要它显示我们会调用
*showAsDropDown(View anchor)* 等类似的方法，这些使用showAsDropDown方法名的方法最终都
会调用同一个方法
```java
public void showAsDropDown(View anchor, int xoff, int yoff, int gravity) {
        if (isShowing() || !hasContentView()) {
            return;
        }

        TransitionManager.endTransitions(mDecorView);

        attachToAnchor(anchor, xoff, yoff, gravity);

        mIsShowing = true;
        mIsDropdown = true;

        final WindowManager.LayoutParams p =
                createPopupLayoutParams(anchor.getApplicationWindowToken());
        preparePopup(p);

        final boolean aboveAnchor = findDropDownPosition(anchor, p, xoff, yoff,
                p.width, p.height, gravity, mAllowScrollingAnchorParent);
        updateAboveAnchor(aboveAnchor);
        p.accessibilityIdOfAnchor =
        (anchor != null) ? anchor.getAccessibilityViewId() : -1;

        invokePopup(p);
    }
```
&emsp;&emsp;现在来一步步分析，首先我们看*attachToAnchor*方法，它的作用是为PopupWindow
添加相应的监听的事件。看看源码：
```java
protected final void attachToAnchor(View anchor, int xoff, int yoff, int gravity) {
        detachFromAnchor();

        final ViewTreeObserver vto = anchor.getViewTreeObserver();
        if (vto != null) {
            vto.addOnScrollChangedListener(mOnScrollChangedListener);
        }
        anchor.addOnAttachStateChangeListener(mOnAnchorDetachedListener);

        final View anchorRoot = anchor.getRootView();
        anchorRoot.addOnAttachStateChangeListener(mOnAnchorRootDetachedListener);
        anchorRoot.addOnLayoutChangeListener(mOnLayoutChangeListener);
        ...
    }
```
&emsp;&emsp;上述方法向*ViewTreeObserver*和需要附着的View和View的RootView添加监听。
但是方法第一行有一个*detachFromAnchor*方法。该方法的作用是为了提高PopupWindow的复用性，
它将移除上一次PopupWindow附着的View所添加的监听所有监听，也包括之前提到的
*ViewTreeObserver* 和View的RootView。

&emsp;&emsp;接着分析最重要的，这是为什么使用*showAsDropDown*方法能够使得PopupWindow
显示在正确位置的主要原因。*LayoutParams* 用来作为View显示位置的依据，而
*createPopupLayoutParams* 方法的作用也就是用来创建对应的LayoutParams属性的。此处就不贴
代码了。

&emsp;&emsp;说了这么多，我们还是没有讲到 *PopupBackgroundView* 和 *PopupDecorView*
是什么时候创建的，现在我们就来分析 *preparePopup(WindowManager.LayoutParams p)* 方法。
```java
private void preparePopup(WindowManager.LayoutParams p) {
        if (mContentView == null || mContext == null || mWindowManager == null) {
            throw new IllegalStateException("You must specify a valid content view by "
                    + "calling setContentView() before attempting to show the popup.");
        }

        // The old decor view may be transitioning out. Make sure it finishes
        // and cleans up before we try to create another one.
        if (mDecorView != null) {
            mDecorView.cancelTransitions();
        }

        // When a background is available, we embed the content view within
        // another view that owns the background drawable.
        if (mBackground != null) {
            mBackgroundView = createBackgroundView(mContentView);
            mBackgroundView.setBackground(mBackground);
        } else {
            mBackgroundView = mContentView;
        }

        mDecorView = createDecorView(mBackgroundView);

        // The background owner should be elevated so that it casts a shadow.
        mBackgroundView.setElevation(mElevation);

        // We may wrap that in another view, so we'll need to manually specify
        // the surface insets.
        p.setSurfaceInsets(mBackgroundView, true /*manual*/, true /*preservePrevious*/);

        mPopupViewInitialLayoutDirectionInherited =
                (mContentView.getRawLayoutDirection() == View.LAYOUT_DIRECTION_INHERIT);
    }
```
&emsp;&emsp;从源码可以看出，*PopupBackgroundView* 使用
*createBackgroundView(View contentView)* 方法创建，*PopupDecorView* 使用
*createDecorView(View contentView)* 创建。两个方法都是通过配置*LayoutParams* 来设置
对应View。
```java
private PopupBackgroundView createBackgroundView(View contentView) {
    final ViewGroup.LayoutParams layoutParams = mContentView.getLayoutParams();
    final int height;
    if (layoutParams != null && layoutParams.height == WRAP_CONTENT) {
        height = WRAP_CONTENT;
    } else {
        height = MATCH_PARENT;
    }

    final PopupBackgroundView backgroundView = new PopupBackgroundView(mContext);
    final PopupBackgroundView.LayoutParams listParams = new PopupBackgroundView.LayoutParams(
            MATCH_PARENT, height);
    backgroundView.addView(contentView, listParams);

    return backgroundView;
}
private PopupDecorView createDecorView(View contentView) {
    final ViewGroup.LayoutParams layoutParams = mContentView.getLayoutParams();
    final int height;
    if (layoutParams != null && layoutParams.height == WRAP_CONTENT) {
        height = WRAP_CONTENT;
    } else {
        height = MATCH_PARENT;
    }

    final PopupDecorView decorView = new PopupDecorView(mContext);
    decorView.addView(contentView, MATCH_PARENT, height);
    decorView.setClipChildren(false);
    decorView.setClipToPadding(false);

    return decorView;
}
```
&emsp;&emsp;根据*preparePopup*、*createBackgroundView* 与 *createDecorView* 的方法知
道，PopupDecorView是PopupBackgroundView的父View。

&emsp;&emsp;使用 *showAsDropDown* 显示PopupWindow在对应View的下方，但是如果相应的View
本身在其父View的底部，那么PopupWindow该如何显示呢？如下代码：
```java
protected final boolean findDropDownPosition(View anchor, WindowManager.LayoutParams outParams,
        int xOffset, int yOffset, int width, int height, int gravity, boolean allowScroll) {
    final int anchorHeight = anchor.getHeight();
    final int anchorWidth = anchor.getWidth();
    if (mOverlapAnchor) {
        yOffset -= anchorHeight;
    }

    // Initially, align to the bottom-left corner of the anchor plus offsets.
    final int[] appScreenLocation = mTmpAppLocation;
    final View appRootView = getAppRootView(anchor);
    appRootView.getLocationOnScreen(appScreenLocation);

    final int[] screenLocation = mTmpScreenLocation;
    anchor.getLocationOnScreen(screenLocation);

    final int[] drawingLocation = mTmpDrawingLocation;
    drawingLocation[0] = screenLocation[0] - appScreenLocation[0];
    drawingLocation[1] = screenLocation[1] - appScreenLocation[1];
    outParams.x = drawingLocation[0] + xOffset;
    outParams.y = drawingLocation[1] + anchorHeight + yOffset;

    final Rect displayFrame = new Rect();
    appRootView.getWindowVisibleDisplayFrame(displayFrame);
    if (width == MATCH_PARENT) {
        width = displayFrame.right - displayFrame.left;
    }
    if (height == MATCH_PARENT) {
        height = displayFrame.bottom - displayFrame.top;
    }

    // Let the window manager know to align the top to y.
    outParams.gravity = computeGravity();
    outParams.width = width;
    outParams.height = height;

    // If we need to adjust for gravity RIGHT, align to the bottom-right
    // corner of the anchor (still accounting for offsets).
    final int hgrav = Gravity.getAbsoluteGravity(gravity, anchor.getLayoutDirection())
            & Gravity.HORIZONTAL_GRAVITY_MASK;
    if (hgrav == Gravity.RIGHT) {
        outParams.x -= width - anchorWidth;
    }

    // First, attempt to fit the popup vertically without resizing.
    final boolean fitsVertical = tryFitVertical(outParams, yOffset, height,
            anchorHeight, drawingLocation[1], screenLocation[1], displayFrame.top,
            displayFrame.bottom, false);

    // Next, attempt to fit the popup horizontally without resizing.
    final boolean fitsHorizontal = tryFitHorizontal(outParams, xOffset, width,
            anchorWidth, drawingLocation[0], screenLocation[0], displayFrame.left,
            displayFrame.right, false);

    // If the popup still doesn't fit, attempt to scroll the parent.
    if (!fitsVertical || !fitsHorizontal) {
        final int scrollX = anchor.getScrollX();
        final int scrollY = anchor.getScrollY();
        final Rect r = new Rect(scrollX, scrollY, scrollX + width + xOffset,
                scrollY + height + anchorHeight + yOffset);
        if (allowScroll && anchor.requestRectangleOnScreen(r, true)) {
            // Reset for the new anchor position.
            anchor.getLocationOnScreen(screenLocation);
            drawingLocation[0] = screenLocation[0] - appScreenLocation[0];
            drawingLocation[1] = screenLocation[1] - appScreenLocation[1];
            outParams.x = drawingLocation[0] + xOffset;
            outParams.y = drawingLocation[1] + anchorHeight + yOffset;

            // Preserve the gravity adjustment.
            if (hgrav == Gravity.RIGHT) {
                outParams.x -= width - anchorWidth;
            }
        }

        // Try to fit the popup again and allowing resizing.
        tryFitVertical(outParams, yOffset, height, anchorHeight, drawingLocation[1],
                screenLocation[1], displayFrame.top, displayFrame.bottom, mClipToScreen);
        tryFitHorizontal(outParams, xOffset, width, anchorWidth, drawingLocation[0],
                screenLocation[0], displayFrame.left, displayFrame.right, mClipToScreen);
    }

    // Return whether the popup's top edge is above the anchor's top edge.
    return outParams.y < drawingLocation[1];
}
```
&emsp;&emsp;从代码可以看出，该方法的作用是调节PopupWindow内部View的位置参数，使其能够
显示在对应的位置。当PopupWindow附着的View位于其父View的底部，并且其父View的底部与屏幕
底部重合，此时按照 *showAsDropDown* 方法最初的规则PopupWindow将会置于底部，当其显示的
时候由于位置在不在屏幕中，因此显示不出，该方法会调节其位置，当附着View的底部剩余空间小于
PopupWindow高度时，那么PopupWindow会显示在附着View的上方，如果这个时候附着View的上部
空间也不够，那么会移动PopupWindow的位置，使其能够在屏幕中显示全，到最后如果出现极端情况，
也就是屏幕全部高度小于PopupWindow高度时，那么会调整PopupWindow的高度。在上面源码中的
*tryFitVertical* 方法就是实现上述功能的。而其后的方法 *tryFitHorizontal* 是调整
*PopupWindow* 在水平方向的显示的，处理方式相似。

&emsp;&emsp;PopupWindow做完这系列的事情之后，就开始更新对应的内部View，做显示的准备了，
做这个功能的就是 *updateAboveAnchor(boolean aboveAnchor) 、invokePopup(WindowManager.LayoutParams p)* 两个方法。

### 开发过程中的深坑
&emsp;&emsp;
坑：showAsDropDown添加的View如果在未绘制之前依附，会有深坑
