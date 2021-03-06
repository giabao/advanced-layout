# Advanced Layout

An easy way to create fluid layouts in Flash and OpenFL. There are two main ways to use it:

1. Define your layout using convenience functions. (See [LayoutUtils](#using-layoututils).)
2. Take a pre-built layout, and make it scale. (See [PremadeLayoutUtils](#using-premadelayoututils).)

If you're switching from OpenFL's Layout library, [click here](#switching-from-openfls-library).

Installation
============

    haxelib install advanced-layout

Using LayoutUtils
=================

To get started, add this after your imports:

    using layout.LayoutUtils;

LayoutUtils is a [static extension](http://haxe.org/manual/lf-static-extension.html), so this isn't required, but it makes things easier.

Scaling
-------

    //Sample objects.
    var mySprite:Sprite = new Sprite();
    mySprite.graphics.beginFill(0x000000);
    mySprite.graphics.drawCircle(40, 40, 40);
    addChild(mySprite);
    var myBitmap:Bitmap = new Bitmap(Assets.getBitmapData("MyBitmap.png"));
    addChild(myBitmap);
    
    //The easiest way to handle scale. The sprite will now scale with
    //the stage.
    mySprite.simpleScale();
    
    //Make the sprite fill half the stage. Caution: this only changes
    //the size; it won't center it.
    mySprite.fillPercentWidth(0.5);
    mySprite.fillPercentHeight(0.5);
    
    //Make the bitmap match the width of the sprite, meaning it will
    //fill half the stage horizontally. The bitmap's height won't
    //scale at all... yet.
    myBitmap.matchWidth(sprite);
    
    //Scale the bitmap's height so that it isn't distorted.
    myBitmap.aspectRatio();

Positioning
-----------

Always set the position after setting the scale.

    //Place the bitmap on the right edge of the stage.
    myBitmap.alignRight();
    
    //Place the sprite in the top-left corner, leaving a small margin.
    mySprite.alignTopLeft(5);
    
    //Place the bitmap 75% of the way down the stage.
    myBitmap.verticalPercent(0.75);
    
    //Place the bitmap a medium distance from the left.
    myBitmap.alignLeft(50);

Sometimes the best way to create a layout is to specify that one object should be above, below, left of, or right of another object.

    //Place the bitmap below the sprite. (Warning: this only affects the
    //y coordinate; the x coordinate will be unchanged.)
    myBitmap.below(mySprite);
    
    //Place the bitmap to the right of the sprite. (This only affects the
    //x coordinate, so now it'll be diagonally below.)
    myBitmap.rightOf(mySprite);
    
    //Center the bitmap directly below the sprite. (This affects both the
    //x and y coordinates.)
    myBitmap.belowCenter(mySprite);
    
    //Place the bitmap directly right of the sprite, a long distance away.
    myBitmap.rightOfCenter(mySprite, 300);
    
    //Place the sprite in the center of the stage. Warning: now the
    //instructions are out of order. The bitmap will be placed to the
    //right of wherever the sprite was last time.
    mySprite.center();
    
    //Correct the ordering.
    myBitmap.rightOfCenter(mySprite, 300);

Conflict resolution
-------------------

Advanced Layout uses an intelligent system to determine whether a given instruction conflicts with a previous one. When a conflict occurs, the old instruction will be removed from the list.

    //The first instruction never conflicts with anything.
    myBitmap.simpleWidth();
    
    //This does not conflict with simpleWidth(), so both will be used.
    myBitmap.fillHeight();
    
    //This conflicts with fillHeight(), so fillHeight() will be replaced.
    myBitmap.simpleHeight();
    
    //This does not conflict with either size command; all three will be used.
    myBitmap.alignLeft();
    
    //This does not conflict with alignLeft() or the size commands, so
    //all of them will be used.
    myBitmap.alignBottom();
    
    //This conflicts with both alignBottom() and alignLeft(), so both
    //will be replaced.
    myBitmap.alignTopCenter();
    
    //This conflicts with only the "top" part of alignTopCenter(), so
    //only that part will be replaced.
    myBitmap.above(mySprite);

It's obviously more efficient not to call all these extra functions in the first place, but if for some reason you have to, it won't cause a memory leak.

Using PremadeLayoutUtils
========================

As the name implies, PremadeLayoutUtils assumes that you've already arranged your layout. All this class does is make your layout scale with the stage.

To get started, add this after your import statements:

    using layout.PremadeLayoutUtils;

PremadeLayoutUtils usage
------------------------

The "fill" functions will cause an object to fill the stage, except for whatever margins the object started with.

    //A sample rectangle, taking up half the stage's width
    //and most of the stage's height.
    var myRect:Shape = new Shape();
    myRect.graphics.beginFill(0x000000);
    myRect.graphics.drawRect(0, 0, stage.stageWidth / 2, stage.stageHeight * 0.8);
    addChild(myRect);
    
    //Center the rectangle horizontally. As the stage's
    //width increases, the rectangle will expand by an equal
    //amount. (The margins will keep their size.)
    myRect.x = stage.stageWidth / 2 - myRect.width / 2;
    myRect.fillWidth();
    
    //Place the rectangle on the bottom, so that it takes
    //up everything except for a small area at the top.
    myRect.y = stage.stageHeight - myRect.height;
    myRect.fillHeight();

When you align an object to an edge, the object's current distance from the edge will be maintained.

    //A sample bitmap.
    var myBitmap:Bitmap = new Bitmap(Assets.getBitmapData("MyBitmap.png"));
    addChild(myBitmap);
    
    //This will cause the bitmap to stay a short distance from
    //the left.
    myBitmap.x = 5;
    myBitmap.alignLeft();
    
    //Aligning the bitmap to the right is not recommended
    //when it's near to the left. If the stage width decreases,
    //the object will be pushed off the left side.
    myBitmap.x = 5;
    myBitmap.alignRight();
    
    //Much better!
    myBitmap.x = stage.stageWidth - myBitmap.width;
    myBitmap.alignRight();

If you align it to the center, its offset from the center will be maintained.

    //Now center it.
    myBitmap.x = stage.stageWidth / 2 - myBitmap.width / 2;
    myBitmap.centerX();
    
    //Put it a short distance left of the center. When the stage
    //scales, the bitmap will stay to the left of the center.
    myBitmap.x = stage.stageWidth / 2 + 50;
    
    //Align it to the bottom, but leave a large margin. On a small
    //stage, it will look like the object is near the center, but
    //on a tall one, the object will appear much closer to the bottom.
    myBitmap.y = stage.stageHeight - 200;
    myBitmap.alignBottom();

Unlike in the LayoutUtils class, all of these functions take care of both positioning and scaling. For instance, `centerY()` takes care of vertical scale, and `alignLeft()` takes care of horizontal scale.

Switching from OpenFL's library
-------------------------------

Most of the old layout code can be removed, except for `layout.addItem()`. Remove anything along these lines:

    import layout.LayoutItem;
    
    private var layout:Layout;
    
    layout = new Layout();
    
    layout.resize(stage.stageWidth, stage.stageHeight);
    
    stage.addEventListener(Event.RESIZE, stage_onResize);

The only thing you need to add is "`using layout.PremadeLayoutUtils;`" after your imports. Next, go through any instances of `layout.addItem()`:

    //Example from SimpleSWFLayout.
    layout.addItem (new LayoutItem (clip.getChildByName ("Background"), STRETCH, STRETCH, false, false));

If the first parameter isn't stored in a variable, separate it out:

    var background:DisplayObject = clip.getChildByName("Background");
    layout.addItem(new LayoutItem(background, STRETCH, STRETCH, false, false));

Next, convert the `verticalPosition` and `horizontalPosition` arguments. Remember, vertical comes first in OpenFL's library:

    //If verticalPosition is STRETCH:
    background.fillHeight();
    
    //If verticalPosition is TOP:
    background.alignTop();
    
    //If verticalPosition is CENTER:
    background.centerY();
    
    //If verticalPosition is BOTTOM:
    background.alignBottom();
    
    //If verticalPosition is NONE, don't do anything.

Horizontal is second:

    //If horizontalPosition is STRETCH:
    background.fillWidth();
    
    //If horizontalPosition is LEFT:
    background.alignLeft();
    
    //If horizontalPosition is CENTER:
    background.centerX();
    
    //If horizontalPosition is RIGHT:
    background.alignRight();
    
    //If horizontalPosition is NONE, don't do anything.

Finally, delete the `layout.addItem()` line, and move on to the next.
