---
layout: post
title: "RecyclerView自动滑动"
date: 2018-03-23
comments: true
categories: android
tags: [RecyclerView]
keywords: RecyclerView 
description:  RecyclerView实现自动滑动
--- 

## RecyclerView实现自动滑动

首先RecyclerView不能手动滑动，需要重写RecyclerView的dispatchTouchEvent方法返回false。
>
	public class NoSlidRecyclerView extends RecyclerView {
> 
>     public NoSlidRecyclerView(Context context) {
>         super(context);
>     }
> 
>     public NoSlidRecyclerView(Context context, @Nullable AttributeSet attrs) {
>         super(context, attrs);
>     }
> 
>     public NoSlidRecyclerView(Context context, @Nullable AttributeSet attrs, int defStyle) {
>         super(context, attrs, defStyle);
>     }
> 
>     @Override
>     public boolean dispatchTouchEvent(MotionEvent ev) {
>         return false;
>     }
>     }

后面通过LinearLayoutManager实现，自定义ScrollSpeedLinearLayoutManger继承LinearLayoutManager

> 
	public class ScrollSpeedLinearLayoutManger extends LinearLayoutManager {
>     private static final String TAG = "ScrollSpeedLinearLayout";
>     //这个值根据实际情况填写，可能快或慢
>     private float MILLISECONDS_PER_INCH = 3f;
>     private Context contxt;
> 
>     public ScrollSpeedLinearLayoutManger(Context context) {
>         super(context);
>         this.contxt = context;
>     }
> 
>     @Override
>     public void smoothScrollToPosition(RecyclerView recyclerView, RecyclerView.State state, int position) {
>         Log.i(TAG, "smoothScrollToPosition: ");
>         LinearSmoothScroller linearSmoothScroller =
>                 new LinearSmoothScroller(recyclerView.getContext()) {
>                     @Override
>                     public PointF computeScrollVectorForPosition(int targetPosition) {
>                         Log.i(TAG, "computeScrollVectorForPosition: ");
>                         return ScrollSpeedLinearLayoutManger.this
>                                 .computeScrollVectorForPosition(targetPosition);
>                     }
> 
>                     //This returns the milliseconds it takes to
>                     //scroll one pixel.
>                     @Override
>                     protected float calculateSpeedPerPixel
>                     (DisplayMetrics displayMetrics) {
>                         Log.i(TAG, "calculateSpeedPerPixel: "+displayMetrics.density);
>                         return MILLISECONDS_PER_INCH * displayMetrics.density;
>                         //返回滑动一个pixel需要多少毫秒
>                     }
> 
>                 };
>         linearSmoothScroller.setTargetPosition(position);
>         startSmoothScroll(linearSmoothScroller);
>      }
>     }
      

然后将自定义的ScrollSpeedLinearLayoutManger设置给RecyclerView
>
	 	ScrollSpeedLinearLayoutManger layoutManger = new ScrollSpeedLinearLayoutManger(this);
>         recyclerView.setLayoutManager(layoutManger);


RecyclerView设置Adapter
>
	TestChildAdapter adapter = new TestChildAdapter(this, list);
    recyclerView.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL));
    recyclerView.setAdapter(adapter);


通过Handler实时判断RecyclerView是否滑到顶部或底部，再重新向下滑或向上滑

>
	private Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            //表示是否能向上滚动，false表示已经滚动到底部
            boolean b = recyclerView.canScrollVertically(1);
            //表示是否能向下滚动，false表示已经滚动到顶部
            boolean b1 = recyclerView.canScrollVertically(-1);
            if (!b) {
                layoutManger.smoothScrollToPosition(recyclerView, null, 0);
            } else if (!b1) {
                layoutManger.smoothScrollToPosition(recyclerView, null, list.size());
            }
            handler.sendEmptyMessageDelayed(0,1000);
        }
    };



完！