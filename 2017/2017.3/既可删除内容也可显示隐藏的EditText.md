#Java文件（依赖于 PasswordEditText）
	/**
	 * ***************************************
	 * author：琚涛
	 * time：2017/3/2
	 * description：显示叉子加眼睛的EditText
	 * ****************************************
	 */
	
	public class PasswordForkEditText extends FrameLayout {
	    private static final int DEFAULT_LENGTH = 20;
	    private PasswordEditText mPet;
	    private ImageView mIv;
	    private String hint;
	    private int maxlength;
	
	    public PasswordForkEditText(Context context) {
	        super(context);
	    }
	
	    public PasswordForkEditText(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        init(attrs, 0);
	    }
	
	    public PasswordForkEditText(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	        init(attrs, defStyleAttr);
	    }
	
	    private void init(AttributeSet attrs, int defStyleAttr) {
	        LayoutInflater.from(getContext()).inflate(R.layout.view_password_fork_edittext, this, true);
	        mPet = (PasswordEditText) findViewById(R.id.pet_password_fork);
	        mIv = (ImageView) findViewById(R.id.iv_password_fork);
	        if (attrs != null) {
	            TypedArray styledAttributes = getContext().getTheme().obtainStyledAttributes(attrs, R.styleable.PasswordForkEditText, defStyleAttr, 0);
	            try {
	                hint = styledAttributes.getString(R.styleable.PasswordForkEditText_hint);
	                maxlength = styledAttributes.getInt(R.styleable.PasswordEditText_pet_iconHide, DEFAULT_LENGTH);
	            } finally {
	                styledAttributes.recycle();
	            }
	        }
	        mPet.setHint(hint);
	        mPet.setFilters(new InputFilter[]{new InputFilter.LengthFilter(maxlength)});
	
	        //添加焦点变化监听
	        mPet.setOnFocusChangeListener(new FocusChangeListenerImpl());
	        //添加文字变化监听
	        mPet.addTextChangedListener(new TextWatcherImpl());
	        //设置右侧图片不可见
	        setClearDrawableVisible(false);
	
	        mIv.setOnTouchListener(new ForkTouchImpl());
	
	    }
	
	    @Override
	    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        mPet.measure(widthMeasureSpec,heightMeasureSpec);
	        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	    }
	
	    public Editable getText() {
	        return mPet.getText();
	    }
	
	    private class FocusChangeListenerImpl implements OnFocusChangeListener {
	
	        private boolean isVisible;
	
	        @Override
	        public void onFocusChange(View view, boolean b) {
	            //获得焦点后,如果内容长度大于1,显示删除图标
	            if (b) {
	                isVisible = mPet.getText().toString().length() > 0;
	            } else {
	                isVisible = false;
	            }
	            setClearDrawableVisible(isVisible);
	
	
	        }
	    }
	
	    private class TextWatcherImpl implements TextWatcher {
	        @Override
	        public void beforeTextChanged(CharSequence charSequence, int i, int i1, int i2) {
	
	        }
	
	        @Override
	        public void onTextChanged(CharSequence charSequence, int i, int i1, int i2) {
	
	        }
	
	        @Override
	        public void afterTextChanged(Editable editable) {
	            //内容改变后,如果长度大于0,显示删除图标
	            boolean isVisible = mPet.getText().toString().length() > 0;
	            setClearDrawableVisible(isVisible);
	        }
	    }
	
	    private class ForkTouchImpl implements OnTouchListener {
	
	        @Override
	        public boolean onTouch(View v, MotionEvent event) {
	            switch (event.getAction()) {
	                case MotionEvent.ACTION_UP:
	                    mPet.setText("");
	                    break;
	                default:
	                    break;
	            }
	            return true;
	        }
	    }
	
	    private void setClearDrawableVisible(boolean isVisible) {
	        if (isVisible) {
	            mIv.setVisibility(VISIBLE);
	        } else {
	            mIv.setVisibility(GONE);
	        }
	
	    }
	
	    public PasswordEditText getEditText() {
	        return mPet;
	    }
	}

#attr文件
    <declare-styleable name="PasswordForkEditText">
        <attr name="hint" format="string"/>
        <attr name="maxLength" format="integer"/>
    </declare-styleable>

#布局文件
	<?xml version="1.0" encoding="utf-8"?>
	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	              android:orientation="vertical"
	              android:layout_width="match_parent"
	              android:layout_height="wrap_content">
	    <com.juexing.tniu.widget.PasswordEditText
	        android:id="@+id/pet_password_fork"
	        android:layout_centerVertical="true"
	        style="@style/CommonEditText"
	        android:layout_width="match_parent"
	        android:layout_height="@dimen/dis_80"
	        android:inputType="textPassword"/>
	
	    <ImageView
	        android:layout_marginTop="@dimen/dis_2"
	        android:padding="@dimen/dis_10"
	        android:id="@+id/iv_password_fork"
	        android:layout_alignParentRight="true"
	        android:layout_centerVertical="true"
	        android:src="@mipmap/fork"
	        android:layout_marginRight="@dimen/dis_40"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"/>
	</RelativeLayout>