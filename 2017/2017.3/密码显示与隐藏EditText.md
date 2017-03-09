# Java文件

	/**
	 * ***************************************
	 * author：琚涛
	 * time：2017/3/1
	 * description：密码显示隐藏编辑框
	 * ****************************************
	 */
	
	public class PasswordEditText extends TextInputEditText {
	
	    private final static int EXTRA_TAPPABLE_AREA = 50;
	
	    @DrawableRes
	    private int showPwIcon = R.mipmap.close_eye;
	
	    @DrawableRes
	    private int hidePwIcon = R.mipmap.open_eye;
	
	    private final static int ALPHA_ICON_ENABLED = (int) (255 * 0.54f);
	
	    private final static int ALPHA_ICON_DISABLED = (int) (255 * 0.38f);
	
	    private Drawable showPwDrawable;
	
	    private Drawable hidePwDrawable;
	
	    private boolean passwordVisible;
	
	    private boolean isRTL;
	
	    private boolean showingIcon;
	
	    private boolean setErrorCalled;
	
	    private boolean hoverShowsPw;
	
	    private boolean useNonMonospaceFont;
	
	    private boolean disableIconAlpha;
	
	    private boolean shouldShowIcon;
	
	    private boolean handlingHoverEvent;
	
	    public PasswordEditText(Context context) {
	        this(context, null);
	    }
	
	    public PasswordEditText(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        initFields(attrs, 0);
	    }
	
	    public PasswordEditText(Context context, AttributeSet attrs, int defStyleAttr) {
	        super(context, attrs, defStyleAttr);
	        initFields(attrs, defStyleAttr);
	    }
	
	    public void initFields(AttributeSet attrs, int defStyleAttr) {
	
	        if (attrs != null) {
	            TypedArray styledAttributes = getContext().getTheme().obtainStyledAttributes(attrs, R.styleable.PasswordEditText, defStyleAttr, 0);
	            try {
	                showPwIcon = styledAttributes.getResourceId(R.styleable.PasswordEditText_pet_iconShow, showPwIcon);
	                hidePwIcon = styledAttributes.getResourceId(R.styleable.PasswordEditText_pet_iconHide, hidePwIcon);
	                hoverShowsPw = styledAttributes.getBoolean(R.styleable.PasswordEditText_pet_hoverShowsPw, false);
	                useNonMonospaceFont = styledAttributes.getBoolean(R.styleable.PasswordEditText_pet_nonMonospaceFont, false);
	                disableIconAlpha = styledAttributes.getBoolean(R.styleable.PasswordEditText_pet_disableIconAlpha, false);
	                shouldShowIcon = styledAttributes.getBoolean(R.styleable.PasswordEditText_pet_shouldshow, true);
	            } finally {
	                styledAttributes.recycle();
	            }
	        }
	
	        hidePwDrawable = ContextCompat.getDrawable(getContext(), hidePwIcon).mutate();
	        showPwDrawable = ContextCompat.getDrawable(getContext(), showPwIcon).mutate();
	
	
	
	        if (!disableIconAlpha) {
	            hidePwDrawable.setAlpha(ALPHA_ICON_ENABLED);
	            showPwDrawable.setAlpha(ALPHA_ICON_DISABLED);
	        }
	
	        if (useNonMonospaceFont) {
	            setTypeface(Typeface.DEFAULT);
	        }
	
	        isRTL = isRTLLanguage();
	        passwordVisible=shouldShowIcon;
	        handlePasswordInputVisibility();
	        showPasswordVisibilityIndicator(shouldShowIcon);
	//        addTextChangedListener(new TextWatcher() {
	//            @Override
	//            public void beforeTextChanged(CharSequence s, int start, int count, int after) {
	//            }
	//
	//            @Override
	//            public void onTextChanged(CharSequence seq, int start, int before, int count) {
	//            }
	//
	//            @Override
	//            public void afterTextChanged(Editable s) {
	//                if (s.length() > 0) {
	//                    if (setErrorCalled) {
	//                        setCompoundDrawables(null, null, null, null);
	//                        setErrorCalled = false;
	//                        showPasswordVisibilityIndicator(true);
	//                    }
	//                    if (!showingIcon) {
	//                        showPasswordVisibilityIndicator(true);
	//                    }
	//                } else {
	//                    passwordVisible = false;
	//                    handlePasswordInputVisibility();
	//                    showPasswordVisibilityIndicator(false);
	//                }
	//
	//            }
	//        });
	    }
	
	    private boolean isRTLLanguage() {
	        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.JELLY_BEAN_MR1) {
	            return false;
	        }
	        Configuration config = getResources().getConfiguration();
	        return config.getLayoutDirection() == View.LAYOUT_DIRECTION_RTL;
	    }
	
	    @Override
	    public Parcelable onSaveInstanceState() {
	        Parcelable superState = super.onSaveInstanceState();
	        return new SavedState(superState, showingIcon, passwordVisible);
	    }
	
	    @Override
	    public void onRestoreInstanceState(Parcelable state) {
	        SavedState savedState = (SavedState) state;
	        super.onRestoreInstanceState(savedState.getSuperState());
	        showingIcon = savedState.isShowingIcon();
	        passwordVisible = savedState.isPasswordVisible();
	        handlePasswordInputVisibility();
	        showPasswordVisibilityIndicator(showingIcon);
	    }
	
	    @Override
	    public void setError(CharSequence error) {
	        super.setError(error);
	        setErrorCalled = true;
	
	    }
	
	    @Override
	    public void setError(CharSequence error, Drawable icon) {
	        super.setError(error, icon);
	        setErrorCalled = true;
	    }
	
	    @Override
	    public boolean onTouchEvent(MotionEvent event) {
	        if (!showingIcon) {
	            return super.onTouchEvent(event);
	        } else {
	            final Rect bounds = showPwDrawable.getBounds();
	            final int x = (int) event.getX();
	            int iconXRect = isRTL ? getLeft() + bounds.width() + EXTRA_TAPPABLE_AREA :
	                    getRight() - bounds.width() - EXTRA_TAPPABLE_AREA;
	
	            switch (event.getAction()) {
	                case MotionEvent.ACTION_DOWN:
	                    if (hoverShowsPw) {
	                        if (isRTL ? x <= iconXRect : x >= iconXRect) {
	                            togglePasswordIconVisibility();
	                            // prevent keyboard from coming up
	                            event.setAction(MotionEvent.ACTION_CANCEL);
	                            handlingHoverEvent = true;
	                        }
	                    }
	                    break;
	                case MotionEvent.ACTION_UP:
	                    if (handlingHoverEvent || (isRTL ? x <= iconXRect : x >= iconXRect)) {
	                        togglePasswordIconVisibility();
	                        event.setAction(MotionEvent.ACTION_CANCEL);
	                        handlingHoverEvent = false;
	                    }
	                    break;
	            }
	            return super.onTouchEvent(event);
	        }
	    }
	
	
	    private void showPasswordVisibilityIndicator(boolean shouldShowIcon) {
	        if (shouldShowIcon) {
	            Drawable drawable = passwordVisible ? hidePwDrawable : showPwDrawable;
	            showingIcon = true;
	            setCompoundDrawablesWithIntrinsicBounds(isRTL ? drawable : null, null, isRTL ? null : drawable, null);
	        } else {
	            // reset drawable
	            setCompoundDrawables(null, null, null, null);
	            showingIcon = false;
	        }
	    }
	
	    private void togglePasswordIconVisibility() {
	        passwordVisible = !passwordVisible;
	        handlePasswordInputVisibility();
	        showPasswordVisibilityIndicator(true);
	    }
	    private void handlePasswordInputVisibility() {
	        int selectionStart = getSelectionStart();
	        int selectionEnd = getSelectionEnd();
	        if (passwordVisible) {
	            setTransformationMethod(null);
	        } else {
	            setTransformationMethod(PasswordTransformationMethod.getInstance());
	
	        }
	        setSelection(selectionStart, selectionEnd);
	
	    }
	
	    protected static class SavedState extends BaseSavedState {
	
	        private final boolean mShowingIcon;
	        private final boolean mPasswordVisible;
	
	        private SavedState(Parcelable superState, boolean sI, boolean pV) {
	            super(superState);
	            mShowingIcon = sI;
	            mPasswordVisible = pV;
	        }
	
	        private SavedState(Parcel in) {
	            super(in);
	            mShowingIcon = in.readByte() != 0;
	            mPasswordVisible = in.readByte() != 0;
	        }
	
	        public boolean isShowingIcon() {
	            return mShowingIcon;
	        }
	
	        public boolean isPasswordVisible() {
	            return mPasswordVisible;
	        }
	
	        @Override
	        public void writeToParcel(Parcel destination, int flags) {
	            super.writeToParcel(destination, flags);
	            destination.writeByte((byte) (mShowingIcon ? 1 : 0));
	            destination.writeByte((byte) (mPasswordVisible ? 1 : 0));
	        }
	
	        public static final Parcelable.Creator<SavedState> CREATOR = new Creator<SavedState>() {
	
	            public SavedState createFromParcel(Parcel in) {
	                return new SavedState(in);
	            }
	
	            public SavedState[] newArray(int size) {
	                return new SavedState[size];
	            }
	
	        };
	    }
	}

# attr文件
    <declare-styleable name="PasswordEditText">
        <attr name="pet_iconShow" format="integer"/>
        <attr name="pet_iconHide" format="integer"/>
        <attr name="pet_hoverShowsPw" format="boolean"/>
        <attr name="pet_nonMonospaceFont" format="boolean"/>
        <attr name="pet_disableIconAlpha" format="boolean"/>
        <attr name="pet_shouldshow" format="boolean"/>
    </declare-styleable>

