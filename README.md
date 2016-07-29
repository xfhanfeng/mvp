# mvp
mvp浅析
M : 还是业务层和模型层
V : 视图层的责任由Activity来担当
P : 新成员Prensenter 用来代理 C(control) 控制层
MVP与MVC最大的不同，其实是Activity职责的变化，由原来的C (控制层) 变成了 V(视图层)，不再管控制层的问题，只管如何去显示。控制层的角色就由我们的新人 Presenter来担当，这种架构就解决了Activity过度耦合控制层和视图层的问题。

具体来看以下Demo：

public class MainActivity extends Activity implements MVPViewInterf,OnItemClickListener{

	private ListView lv;
	private ProgressBar pb;
	private MVPPresenter presenter;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		initView();
	}
	
	@Override
	protected void onResume() {
		super.onResume();
		presenter.onResum();
	}
	
	public void initView(){
		lv = (ListView) findViewById(R.id.lv);
		pb = (ProgressBar) findViewById(R.id.pb);
		presenter = new MVPPresenter(this);
	}

	@Override
	public void showProgressView() {
		pb.setVisibility(View.VISIBLE);
	}

	@Override
	public void hideProgressView() {
		pb.setVisibility(View.GONE);
	}

	@Override
	public void setListItem(ArrayList<String> list) {
		ArrayAdapter adapter=new ArrayAdapter(MainActivity.this, android.R.layout.simple_expandable_list_item_1, list);
		lv.setAdapter(adapter);
		lv.setOnItemClickListener(this);
		
	}

	@Override
	public void showMessages(String str) {
		Toast.makeText(this, str, Toast.LENGTH_LONG).show();
	}

	@Override
	public void onItemClick(AdapterView<?> parent, View view, int position,
			long id) {
		presenter.onItemClick("item"+position+"被点击了");
	}

	
}



public interface MVPLoadListener {
	public void onSuccess(ArrayList<String> all);
	public void onFailed();

}


public interface MVPViewInterf {
	//显示加载进度视图
	public void showProgressView();
	//隐藏加载进度视图
	public void hideProgressView();
	//初始化ListView
	public void setListItem(ArrayList<String> list);
	//显示信息
	public void showMessages(String str);

}


public class MVPModel {
	public void loadData(final MVPLoadListener listener){
		new Thread(new Runnable() {
			
			@Override
			public void run() {
				try {
					Thread.sleep(3000);
					ArrayList<String> list=new ArrayList<String>();
					for (int i = 0; i < 8; i++) {
						list.add("item"+i);
					}
					if(listener!=null){
						listener.onSuccess(list);
					}
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				
			}
		}).start();
	}

}

public class MVPPresenter {
	private MVPViewInterf mvpView;
	private MVPModel model;
	private Handler mHandler;
	public MVPPresenter(MVPViewInterf mvpView){
		this.mvpView=mvpView;
		model=new MVPModel();
		mHandler=new Handler(Looper.getMainLooper());
	}
	
	public void onResum(){
		mvpView.showProgressView();
		model.loadData(new MVPLoadListener() {
			
			@Override
			public void onSuccess(final ArrayList<String> all) {
				//由于请求开启了新的线程，所以更新界面应提交给主线程
				mHandler.post(new Runnable() {
					
					@Override
					public void run() {
						mvpView.hideProgressView();
						mvpView.setListItem(all);
					}
				});
				
			}
			
			@Override
			public void onFailed() {
				mvpView.hideProgressView();
				mvpView.showMessages("数据请求失败");
			}
		});
	}
	
	public void onItemClick(String str){
		mvpView.showMessages(str);
	}

}
