	/**
	 * ***************************************
	 * author：琚涛
	 * time：2017/3/1
	 * description：用来监听一个多个EditText是否有数据以改变指定Button状态
	 * ****************************************
	 */
	
	public class EditCheckManager extends SimpleTextWatcher {
	
	    private EditText[] mEditTexts;
	    private Button mBtn;
	
	    private EditCheckManager() {
	    }
	
	    private EditCheckManager(Button btn, EditText... editTexts) {
	        mEditTexts = editTexts;
	        mBtn = btn;
	    }
	
	    public static EditCheckManager create(Button btn, EditText... editTexts) {
	
	        return new EditCheckManager(btn, editTexts);
	    }
	
	    public void listener() {
	        if (mBtn == null || mEditTexts == null) {
	            return;
	        }
	
	        for (EditText editText : mEditTexts) {
	            editText.addTextChangedListener(this);
	        }
	
	        mBtn.setEnabled(checkAllEdit());
	    }
	
	    public void removeListener(){
	        for (EditText editText : mEditTexts) {
	            editText.removeTextChangedListener(this);
	        }
	    }
	    @Override
	    public void onTextChanged(CharSequence s, int start, int before, int count) {
	        mBtn.setEnabled(checkAllEdit());
	    }
	
	    private boolean checkAllEdit() {
	        for (EditText editText : mEditTexts) {
	            if (TextUtils.isEmpty(editText.getText() + "")) {
	                return false;
	            }
	        }
	        return true;
	    }
	}


调用方式

	EditCheckManager.create(button,edittext1,edittext2,edittext3,....).listener();