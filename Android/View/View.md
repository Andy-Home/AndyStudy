# View
### LayoutParams(android 26)
&emsp;&emsp;在Android中所有原生的LayoutParams都继承自*ViewGroup.LayoutParams*,贴一
波源码：
```java
/**
 * LayoutParams are used by vinfgv  ews to tell their parents how they want to be
 * laid out. See
 * {@link android.R.styleable#ViewGroup_Layout ViewGroup Layout Attributes}
 * for a list of all child view attributes that this class supports.
 *
 * <p>
 * The base LayoutParams class just describes how big the view wants to be
 * for both width and height. For each dimension, it can specify one of:
 * <ul>
 * <li>FILL_PARENT (renamed MATCH_PARENT in API Level 8 and higher), which
 * means that the view wants to be as big as its parent (minus padding)
 * <li> WRAP_CONTENT, which means that the view wants to be just big enough
 * to enclose its content (plus padding)
 * <li> an exact number
 * </ul>
 * There are subclasses of LayoutParams for different subclasses of
 * ViewGroup. For example, AbsoluteLayout has its own subclass of
 * LayoutParams which adds an X and Y value.</p>
 *
 * <div class="special reference">
 * <h3>Developer Guides</h3>
 * <p>For more information about creating user interface layouts, read the
 * <a href="{@docRoot}guide/topics/ui/declaring-layout.html">XML Layouts</a> developer
 * guide.</p></div>
 *
 * @attr ref android.R.styleable#ViewGroup_Layout_layout_height
 * @attr ref android.R.styleable#ViewGroup_Layout_layout_width
 */
public static class LayoutParams {
    /**
     * Special value for the height or width requested by a View.
     * FILL_PARENT means that the view wants to be as big as its parent,
     * minus the parent's padding, if any. This value is deprecated
     * starting in API Level 8 and replaced by {@link #MATCH_PARENT}.
     */
    @SuppressWarnings({"UnusedDeclaration"})
    @Deprecated
    public static final int FILL_PARENT = -1;

    /**
     * Special value for the height or width requested by a View.
     * MATCH_PARENT means that the view wants to be as big as its parent,
     * minus the parent's padding, if any. Introduced in API Level 8.
     */
    public static final int MATCH_PARENT = -1;

    /**
     * Special value for the height or width requested by a View.
     * WRAP_CONTENT means that the view wants to be just large enough to fit
     * its own internal content, taking its own padding into account.
     */
    public static final int WRAP_CONTENT = -2;

    /**
     * Information about how wide the view wants to be. Can be one of the
     * constants FILL_PARENT (replaced by MATCH_PARENT
     * in API Level 8) or WRAP_CONTENT, or an exact size.
     */
    @ViewDebug.ExportedProperty(category = "layout", mapping = {
        @ViewDebug.IntToString(from = MATCH_PARENT, to = "MATCH_PARENT"),
        @ViewDebug.IntToString(from = WRAP_CONTENT, to = "WRAP_CONTENT")
    })
    public int width;

    /**
     * Information about how tall the view wants to be. Can be one of the
     * constants FILL_PARENT (replaced by MATCH_PARENT
     * in API Level 8) or WRAP_CONTENT, or an exact size.
     */
    @ViewDebug.ExportedProperty(category = "layout", mapping = {
        @ViewDebug.IntToString(from = MATCH_PARENT, to = "MATCH_PARENT"),
        @ViewDebug.IntToString(from = WRAP_CONTENT, to = "WRAP_CONTENT")
    })
    public int height;

    /**
     * Used to animate layouts.
     */
    public LayoutAnimationController.AnimationParameters layoutAnimationParameters;

    /**
     * Creates a new set of layout parameters. The values are extracted from
     * the supplied attributes set and context. The XML attributes mapped
     * to this set of layout parameters are:
     *
     * <ul>
     *   <li><code>layout_width</code>: the width, either an exact value,
     *   {@link #WRAP_CONTENT}, or {@link #FILL_PARENT} (replaced by
     *   {@link #MATCH_PARENT} in API Level 8)</li>
     *   <li><code>layout_height</code>: the height, either an exact value,
     *   {@link #WRAP_CONTENT}, or {@link #FILL_PARENT} (replaced by
     *   {@link #MATCH_PARENT} in API Level 8)</li>
     * </ul>
     *
     * @param c the application environment
     * @param attrs the set of attributes from which to extract the layout
     *              parameters' values
     */
    public LayoutParams(Context c, AttributeSet attrs) {
        TypedArray a = c.obtainStyledAttributes(attrs, R.styleable.ViewGroup_Layout);
        setBaseAttributes(a,
                R.styleable.ViewGroup_Layout_layout_width,
                R.styleable.ViewGroup_Layout_layout_height);
        a.recycle();
    }

    /**
     * Creates a new set of layout parameters with the specified width
     * and height.
     *
     * @param width the width, either {@link #WRAP_CONTENT},
     *        {@link #FILL_PARENT} (replaced by {@link #MATCH_PARENT} in
     *        API Level 8), or a fixed size in pixels
     * @param height the height, either {@link #WRAP_CONTENT},
     *        {@link #FILL_PARENT} (replaced by {@link #MATCH_PARENT} in
     *        API Level 8), or a fixed size in pixels
     */
    public LayoutParams(int width, int height) {
        this.width = width;
        this.height = height;
    }

    /**
     * Copy constructor. Clones the width and height values of the source.
     *
     * @param source The layout params to copy from.
     */
    public LayoutParams(LayoutParams source) {
        this.width = source.width;
        this.height = source.height;
    }

    /**
     * Used internally by MarginLayoutParams.
     * @hide
     */
    LayoutParams() {
    }

    /**
     * Extracts the layout parameters from the supplied attributes.
     *
     * @param a the style attributes to extract the parameters from
     * @param widthAttr the identifier of the width attribute
     * @param heightAttr the identifier of the height attribute
     */
    protected void setBaseAttributes(TypedArray a, int widthAttr, int heightAttr) {
        width = a.getLayoutDimension(widthAttr, "layout_width");
        height = a.getLayoutDimension(heightAttr, "layout_height");
    }

    /**
     * Resolve layout parameters depending on the layout direction. Subclasses that care about
     * layoutDirection changes should override this method. The default implementation does
     * nothing.
     *
     * @param layoutDirection the direction of the layout
     *
     * {@link View#LAYOUT_DIRECTION_LTR}
     * {@link View#LAYOUT_DIRECTION_RTL}
     */
    public void resolveLayoutDirection(int layoutDirection) {
    }

    /**
     * Returns a String representation of this set of layout parameters.
     *
     * @param output the String to prepend to the internal representation
     * @return a String with the following format: output +
     *         "ViewGroup.LayoutParams={ width=WIDTH, height=HEIGHT }"
     *
     * @hide
     */
    public String debug(String output) {
        return output + "ViewGroup.LayoutParams={ width="
                + sizeToString(width) + ", height=" + sizeToString(height) + " }";
    }

    /**
     * Use {@code canvas} to draw suitable debugging annotations for these LayoutParameters.
     *
     * @param view the view that contains these layout parameters
     * @param canvas the canvas on which to draw
     *
     * @hide
     */
    public void onDebugDraw(View view, Canvas canvas, Paint paint) {
    }

    /**
     * Converts the specified size to a readable String.
     *
     * @param size the size to convert
     * @return a String instance representing the supplied size
     *
     * @hide
     */
    protected static String sizeToString(int size) {
        if (size == WRAP_CONTENT) {
            return "wrap-content";
        }
        if (size == MATCH_PARENT) {
            return "match-parent";
        }
        return String.valueOf(size);
    }

    /** @hide */
    void encode(@NonNull ViewHierarchyEncoder encoder) {
        encoder.beginObject(this);
        encodeProperties(encoder);
        encoder.endObject();
    }

    /** @hide */
    protected void encodeProperties(@NonNull ViewHierarchyEncoder encoder) {
        encoder.addProperty("width", width);
        encoder.addProperty("height", height);
    }
}

```
&emsp;&emsp;从源码看出，LayoutParams有两个基础属性变量，一个对象属性变量，三个基础属性
常量。三个基础属性常量是在布局中常使用的：
```java
public static final int FILL_PARENT = -1;
public static final int MATCH_PARENT = -1;
public static final int WRAP_CONTENT = -2;
```
&emsp;&emsp;可以发现其中 FILL_PARENT 与 MATCH_PARENT 的值是相同的，这是因为从API 8
开始，FILL_PARENT 就被废弃，替代它功能的就是 MATCH_PARENT，这样的话它们两个的值相同也
是能够理解的了。

&emsp;&emsp;再来看看基础属性变量：
```java
public int width;
public int height;
```
&emsp;&emsp;同样的，这也是最基础的属性，用来定义View的宽与高。最后一个是对象属相变量：
```java
public LayoutAnimationController.AnimationParameters layoutAnimationParameters;
```
该对象属性的作用是处理动画，对动画的布局位置的操作。

&emsp;&emsp;仔细看过源码的小童鞋可以发现，LayoutParams的源码中有两个方法是空的，来看看：
```java
public void resolveLayoutDirection(int layoutDirection) {}
public void onDebugDraw(View view, Canvas canvas, Paint paint) {}
```
