# RecyclerView 



## 缓存机制

[参考]: https://www.jianshu.com/p/3e9aa4bdaefd?utm_source=desktop&amp;amp;utm_medium=timeline

### ListView 缓存机制

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gho5rz4jqqj30u00v1409.jpg)

ListView的缓存有两级，在ListView里面有一个内部类 RecycleBin，RecycleBin有两个对象Active View和Scrap View来管理缓存，Active View是第一级，Scrap View是第二级。

- **Active View**：是缓存在屏幕内的ItemView，当列表数据发生变化时，屏幕内的数据可以直接拿来复用，无须进行数据绑定。
- **Scrap view**：缓存屏幕外的ItemView，这里所有的缓存的数据都是"脏的"，也就是数据需要重新绑定，也就是说屏幕外的所有数据在进入屏幕的时候都要走一遍getView（）方法。

ListView的缓存流程（ListView 缓存的是 ItemView）：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gho5yotrjij30nw13k408.jpg)

### RecyclerView的缓存机制

RecyclerView的缓存分为四级

- **Scrap**  

  对应ListView 的Active View，屏幕内的缓存数据，可以直接拿来复用。

  Scrap缓存列表（mChangedScrap、mAttachedScrap）是RecyclerView最先查找ViewHolder地方，它跟RecyclerViewPool或者ViewCache有很大的区别。

  mChangedScrap和mAttachedScrap只在布局阶段使用。其他时候它们是空的。布局完成之后，这两个缓存中的viewHolder，会移到mCacheView或者 RecyclerViewPool中。

  当LayoutManager开始布局的时候（预布局或者是最终布局），当前布局中的所有view，都会被dump到scrap中（具体实现可见LinearLayoutManager#onLayoutChildren()方法中调用了detachAndScrapAttachedViews() ），然后LayoutManager挨个地取回view，除非view发生了什么变化，否则它会马上从scrap中回到原来的位置。

  除了在布局时不为空外，还有另一个与scrap有关的规律：所有scrap的 view 都会跟RecyclerView分离。ViewGroup中的`attachView`和`detachView`方法跟addView和removeView方法很像，但是**不会触发请求布局会重绘的事件**。它们只是从ViewGroup的子view列表中删除对应的子view，并将该子view的parent设置为null。detached状态必须是临时，后面紧随着attach或者remove事件。

  

- **Cache**

  **最近**移出屏幕的缓存数据，**默认大小是2个**。当其容量被充满同时又有新的数据添加的时候，会根据FIFO原则，把优先进入的缓存数据移出并放到下一级缓存中，然后再把新的数据添加进来。Cache里面的数据是干净的，也就是携带了原来的ViewHolder的所有数据信息，数据可以直接来拿来复用。需要注意的是，**cache是根据position来寻找数据的**，这个postion是根据第一个或者最后一个可见的item的position以及用户操作行为（上拉还是下拉）。

  

  举个**栗子**：当前屏幕内第一个可见的item的position是1，用户进行了一个下拉操作，那么当前预测的position就相当于（1-1=0），也就是**position=0**的那个item要被拉回到屏幕，此时RecyclerView就从Cache里面找position=0的数据，如果找到了就直接拿来复用。

- **ViewCacheExtension**

  需要自定义，不明了，慎用

- **RecycledViewPool**

  Cache满了后，会格局FIFO原则，放入RecycledViewPool中，RecycledViewPool**默认的缓存数量是5个**。此处不同的是，放入RecycledViewPool前，**ViewHolder会被重置**。**RecycledViewPool是根据itemType**获取的，RecycledViewPool取出的Item需要重新进行数据绑定（`onBindViewHolder()`）。



![](https://tva1.sinaimg.cn/large/007S8ZIlgy1gho698jcggj30vx0u0mzg.jpg)

获取流程图：

![](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghozpl64eaj30rg1d40v9.jpg)



一般情况下(忽略ViewCacheExtension，因为这个需要自己实现)，它有4个存放回收Holder的集合，分别是：

- 可直接重用的临时缓存：mAttachedScrap，mChangedScrap；
- 可直接重用的缓存：mCachedViews；
- 需重新绑定数据的缓存：mRecyclerPool.mScrap；

**为什么说前面两个是临时缓存呢？**
因为每当RecyclerView的dispatchLayout方法结束之前（当调用RecyclerView的reuqestLayout方法或者调用Adapter的一系列notify方法会回调这个dispatchLayout），它们里面的Holder都会移动到mCachedViews或mRecyclerPool.mScrap中。

**那为什么有两个呢？它们之间有什么区别吗？**
它们之间的区别就是：mChangedScrap只能在预布局状态下重用，因为它里面装的都是即将要放到mRecyclerPool中的Holder，而mAttachedScrap则可以在非预布局状态下重用。

**什么是预布局(PreLayout)？**
顾名思义，就是在真正布局之前，事先布局一次。但在预布局状态下，应该把已经remove掉的Item也layout出来，我们可以通过ViewHolder的LayoutParams.isViewRemoved()方法来判断这个ViewHolder是否已经被remove掉。
只有在Adapter的数据集更新时，并且调用的是除notifyDataSetChanged以外的一系列notify方法，预布局才会生效。这也是为什么调用notifyDataSetChanged方法不会播放Item动画的原因了。
这个其实有点像加载Bitmap的操作：先设置只读边，等获取到图片尺寸后设置好缩放比例再真正把图片加载进来。
要开启预布局的话，需要重写LayoutManager中的**supportsPredictiveItemAnimations**方法并**return true;** 这样就能生效了(当然，自带的那三个LayoutManager已经是开启了这个效果的)，当Adapter的数据集更新时，onLayoutChildren方法就会回调两次，第一次是预布局，第二次是真实的布局，我们也可以通过**state.isPreLayout()** 来判断当前是否为预布局状态，并根据这个状态来决定要layout的Item。

> 此处引用：https://blog.csdn.net/u011387817/article/details/81875021



**源码分析**


Recycler RecyclerView 的内部类：

```java
public final class Recycler {
  //LayoutManager每次layout子View之前，那些已经添加到RecyclerView中的Item以及被删除的Item的临时存放地。
        final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
  // 存放可见范围内有更新的Item。
        ArrayList<ViewHolder> mChangedScrap = null;
// 存放滚动过程中没有被重新使用且状态无变化的那些旧Item。
        final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();

        private final List<ViewHolder>
                mUnmodifiableAttachedScrap = Collections.unmodifiableList(mAttachedScrap);

        private int mRequestedCacheMax = DEFAULT_CACHE_SIZE;
  // 默认缓存个数
        int mViewCacheMax = DEFAULT_CACHE_SIZE;

        RecycledViewPool mRecyclerPool;

        private ViewCacheExtension mViewCacheExtension;

        static final int DEFAULT_CACHE_SIZE = 2;
        ...
        }
```

`RecyclerView.RecyclerViewPool`:

```java
 public static class RecycledViewPool {
        private static final int DEFAULT_MAX_SCRAP = 5;
        static class ScrapData {
            final ArrayList<ViewHolder> mScrapHeap = new ArrayList<>();
            int mMaxScrap = DEFAULT_MAX_SCRAP;
            long mCreateRunningAverageNs = 0;
            long mBindRunningAverageNs = 0;
        }
        SparseArray<ScrapData> mScrap = new SparseArray<>();
        ...
        }
```

入口是`androidx.recyclerview.widget.LayoutState#next`方法：

```java
 /**
     * Gets the view for the next element that we should render.
     * Also updates current item index to the next item, based on {@link #mItemDirection}
     *
     * @return The next element that we should render.
     */
    View next(RecyclerView.Recycler recycler) {
        final View view = recycler.getViewForPosition(mCurrentPosition);
        mCurrentPosition += mItemDirection;
        return view;
    }
```

调用了recycler中的通过position获取ItemView：

```java
			 @NonNull
        public View getViewForPosition(int position) {
            return getViewForPosition(position, false);
        }

        View getViewForPosition(int position, boolean dryRun) {
            return tryGetViewHolderForPositionByDeadline(position, dryRun, FOREVER_NS).itemView;
        }
```

```java
 @Nullable
        ViewHolder tryGetViewHolderForPositionByDeadline(int position,
                boolean dryRun, long deadlineNs) {
            if (position < 0 || position >= mState.getItemCount()) {
                throw new IndexOutOfBoundsException("Invalid item position " + position
                        + "(" + position + "). Item count:" + mState.getItemCount()
                        + exceptionLabel());
            }
            boolean fromScrapOrHiddenOrCache = false;
            ViewHolder holder = null;
            // 0) If there is a changed scrap, try to find from there
            if (mState.isPreLayout()) {
                holder = getChangedScrapViewForPosition(position);
                fromScrapOrHiddenOrCache = holder != null;
            }
            // 1) Find by position from scrap/hidden list/cache
            if (holder == null) {
                holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun);
                if (holder != null) {
                    if (!validateViewHolderForOffsetPosition(holder)) {
                        // recycle holder (and unscrap if relevant) since it can't be used
                        if (!dryRun) {
                            // we would like to recycle this but need to make sure it is not used by
                            // animation logic etc.
                            holder.addFlags(ViewHolder.FLAG_INVALID);
                            if (holder.isScrap()) {
                                removeDetachedView(holder.itemView, false);
                                holder.unScrap();
                            } else if (holder.wasReturnedFromScrap()) {
                                holder.clearReturnedFromScrapFlag();
                            }
                            recycleViewHolderInternal(holder);
                        }
                        holder = null;
                    } else {
                        fromScrapOrHiddenOrCache = true;
                    }
                }
            }
            if (holder == null) {
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                if (offsetPosition < 0 || offsetPosition >= mAdapter.getItemCount()) {
                    throw new IndexOutOfBoundsException("Inconsistency detected. Invalid item "
                            + "position " + position + "(offset:" + offsetPosition + ")."
                            + "state:" + mState.getItemCount() + exceptionLabel());
                }

                final int type = mAdapter.getItemViewType(offsetPosition);
                // 2) Find from scrap/cache via stable ids, if exists
                if (mAdapter.hasStableIds()) {
                    holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition),
                            type, dryRun);
                    if (holder != null) {
                        // update position
                        holder.mPosition = offsetPosition;
                        fromScrapOrHiddenOrCache = true;
                    }
                }
                if (holder == null && mViewCacheExtension != null) {
                    // We are NOT sending the offsetPosition because LayoutManager does not
                    // know it.
                    final View view = mViewCacheExtension
                            .getViewForPositionAndType(this, position, type);
                    if (view != null) {
                        holder = getChildViewHolder(view);
                        if (holder == null) {
                            throw new IllegalArgumentException("getViewForPositionAndType returned"
                                    + " a view which does not have a ViewHolder"
                                    + exceptionLabel());
                        } else if (holder.shouldIgnore()) {
                            throw new IllegalArgumentException("getViewForPositionAndType returned"
                                    + " a view that is ignored. You must call stopIgnoring before"
                                    + " returning this view." + exceptionLabel());
                        }
                    }
                }
                if (holder == null) { // fallback to pool
                    if (DEBUG) {
                        Log.d(TAG, "tryGetViewHolderForPositionByDeadline("
                                + position + ") fetching from shared pool");
                    }
                  // 根据type 从RecycledViewPool 获取ViewHolder
                    holder = getRecycledViewPool().getRecycledView(type);
                    if (holder != null) {
                      // 重置ViewHolder
                        holder.resetInternal();
                        if (FORCE_INVALIDATE_DISPLAY_LIST) {
                            invalidateDisplayListInt(holder);
                        }
                    }
                }
                if (holder == null) {
                    long start = getNanoTime();
                    if (deadlineNs != FOREVER_NS
                            && !mRecyclerPool.willCreateInTime(type, start, deadlineNs)) {
                        // abort - we have a deadline we can't meet
                        return null;
                    }
                  // 没有缓存符合，创建一个
                    holder = mAdapter.createViewHolder(RecyclerView.this, type);
                    if (ALLOW_THREAD_GAP_WORK) {
                        // only bother finding nested RV if prefetching
                        RecyclerView innerView = findNestedRecyclerView(holder.itemView);
                        if (innerView != null) {
                            holder.mNestedRecyclerView = new WeakReference<>(innerView);
                        }
                    }

                    long end = getNanoTime();
                    mRecyclerPool.factorInCreateTime(type, end - start);
                    if (DEBUG) {
                        Log.d(TAG, "tryGetViewHolderForPositionByDeadline created new ViewHolder");
                    }
                }
            }

            // This is very ugly but the only place we can grab this information
            // before the View is rebound and returned to the LayoutManager for post layout ops.
            // We don't need this in pre-layout since the VH is not updated by the LM.
            if (fromScrapOrHiddenOrCache && !mState.isPreLayout() && holder
                    .hasAnyOfTheFlags(ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST)) {
                holder.setFlags(0, ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);
                if (mState.mRunSimpleAnimations) {
                    int changeFlags = ItemAnimator
                            .buildAdapterChangeFlagsForAnimations(holder);
                    changeFlags |= ItemAnimator.FLAG_APPEARED_IN_PRE_LAYOUT;
                    final ItemHolderInfo info = mItemAnimator.recordPreLayoutInformation(mState,
                            holder, changeFlags, holder.getUnmodifiedPayloads());
                    recordAnimationInfoIfBouncedHiddenView(holder, info);
                }
            }

            boolean bound = false;
            if (mState.isPreLayout() && holder.isBound()) {
                // do not update unless we absolutely have to.
                holder.mPreLayoutPosition = position;
            } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {
                if (DEBUG && holder.isRemoved()) {
                    throw new IllegalStateException("Removed holder should be bound and it should"
                            + " come here only in pre-layout. Holder: " + holder
                            + exceptionLabel());
                }
                final int offsetPosition = mAdapterHelper.findPositionOffset(position);
                bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs);
            }

            final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
            final LayoutParams rvLayoutParams;
            if (lp == null) {
                rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else if (!checkLayoutParams(lp)) {
                rvLayoutParams = (LayoutParams) generateLayoutParams(lp);
                holder.itemView.setLayoutParams(rvLayoutParams);
            } else {
                rvLayoutParams = (LayoutParams) lp;
            }
            rvLayoutParams.mViewHolder = holder;
            rvLayoutParams.mPendingInvalidate = fromScrapOrHiddenOrCache && bound;
            return holder;
        }
```



[预布局、预测动画、缓存优化建议]: https://mp.weixin.qq.com/s/Kb9GnfpdDjxvt4HUpG-GSw

