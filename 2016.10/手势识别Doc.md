    mDetector=new GestureDetector(new GestureDetector.OnGestureListener() {
      /**
       * 手指按下执行一次
       * @param e
       */
      @Override public boolean onDown(MotionEvent e) {
        return false;
      }

      /**
       * 短暂的按下操作
       * @param e
       */
      @Override public void onShowPress(MotionEvent e) {

      }

      /**
       * 单击操作
       * @param motionEvent
       * @return
       */
      @Override public boolean onSingleTapUp(MotionEvent motionEvent) {
        return false;
      }

      /**
       * 滚动监听
       * @param e1 手指按下事件
       * @param e2 当前的move事件
       * @param distanceX  上一次move事件的x - 当前move事件的x
       * @param distanceY 上一次move事件的y - 当前move事件的y
       * @return
       */
      @Override public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX,
          float distanceY) {
        return false;
      }

      /**
       * 长按
       * @param e
       */
      @Override public void onLongPress(MotionEvent e) {

      }

      /**
       * 手指离开屏幕，控件惯性效果时
       * @param e1 手指按下的事件
       * @param e2 当前的手指move事件
       * @param velocityX 当手指松开的瞬间，x轴的移动速率 像素/秒
       * @param velocityY 当手指松开的瞬间，y轴的移动速率 像素/秒
       * @return
       */
      @Override
      public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
        return false;
      }
    });