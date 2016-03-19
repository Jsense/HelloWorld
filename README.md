package com.smsnotify;

import java.util.ArrayList;
import java.util.List;

import android.annotation.SuppressLint;
import android.content.Intent;
import android.graphics.Color;
import android.graphics.drawable.BitmapDrawable;
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;
import android.support.v4.view.ViewPager;
import android.view.LayoutInflater;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.ImageView;
import android.widget.LinearLayout;
import android.widget.TextView;
import android.widget.LinearLayout.LayoutParams;
import android.widget.PopupWindow;

import com.smsnotify.data.DataBaseOperation;
import com.smsnotify.util.Common;
import com.smsnotify.widget.ViewPagerIconMenu;


public class MainActivity extends FragmentActivity{

	public static final int FRAGMENT_PAGER_0=0;
	public static final int FRAGMENT_PAGER_1=1;
	public static final int FRAGMENT_PAGER_2=2;

	private Common common;
	private TextView serviceButton;
	private ImageView menuImg;

	ViewPager viewPager;
	ViewPagerIconMenu viewPagerIconMenu;
	int[][] icons={{R.drawable.test3_1,R.drawable.test3_2},
			{R.drawable.test1_1,R.drawable.test1_2},
			{R.drawable.test2_1,R.drawable.test2_2}};
	String[] tabNames={"消息","短信回复","黑名单"};

	MsgFragment msgFragment;
	ContentFragment contentFragment;
	HangupFragment hangupFragment;

	List<Fragment> listFragments;
	
	private int serviceState;
	private final int SERVICE_START=0;
	private final int SERVICE_STOP=1;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.main);

		common=new Common(this);
		DataBaseOperation.init(MainActivity.this);

		listFragments=new ArrayList<Fragment>();
		msgFragment=new MsgFragment(this);
		contentFragment=new ContentFragment(this);
		hangupFragment=new HangupFragment(this);

		listFragments.add(msgFragment);
		listFragments.add(contentFragment);
		listFragments.add(hangupFragment);

		viewPager=(ViewPager)findViewById(R.id.viewpager);
		viewPager.setAdapter(new MFragmentPagerAdapter(getSupportFragmentManager()));


		viewPagerIconMenu=(ViewPagerIconMenu)findViewById(R.id.viewpagerTab);
		viewPagerIconMenu.setViewPager(this, viewPager, icons, tabNames);
		
		init();
	}

	@Override
	protected void onRestart() {
		super.onRestart();
		contentFragment.updateListView();
		hangupFragment.updateListView();
	}

	@Override
	protected void onActivityResult(int arg0, int arg1, Intent arg2) {
		super.onActivityResult(arg0, arg1, arg2);
		
		switch (arg1) {
		case 0://ViewPager显示的Fragment
		int test=arg2.getIntExtra("pager", -1);
		if(test!=-1){
			viewPager.setCurrentItem(arg2.getIntExtra("pager", FRAGMENT_PAGER_0));
		}
		break;
		}
	}

	public void init(){

		serviceButton=(TextView)findViewById(R.id.main_service);
		serviceButton.setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				if(serviceButton.getTextColors().getDefaultColor()==Color.BLACK){
					serviceButton.setTextColor(Color.rgb(45, 190, 96));//2DBE60
//					Intent intent=new Intent();
//					intent.setClass(MainActivity.this, PhoneService.class);
//					startService(intent);
				}else{
					serviceButton.setTextColor(Color.BLACK);
//					Intent intent=new Intent();
//					intent.setClass(MainActivity.this, PhoneService.class);
//					stopService(intent);
				}
				
			}
		});
		
		menuImg=(ImageView)findViewById(R.id.main_menu);
		menuImg.setOnClickListener(new OnClickListener() {
			@SuppressLint("InflateParams") 
			@Override
			public void onClick(View v) {
				View popupWindowLayout=LayoutInflater.from(MainActivity.this)
						.inflate(R.layout.menu_popup, null);
				LinearLayout addRule=(LinearLayout)popupWindowLayout.findViewById(R.id.menu_popup_addrule);

				final PopupWindow popupWindow=new PopupWindow(popupWindowLayout,
						LayoutParams.WRAP_CONTENT,LayoutParams.WRAP_CONTENT, true);
				popupWindow.setBackgroundDrawable(new BitmapDrawable());

				addRule.setOnClickListener(new OnClickListener() {
					@Override
					public void onClick(View v) {
						popupWindow.dismiss();

						Intent intent=new Intent();
						intent.putExtra("operationType", 0);
						intent.setClass(MainActivity.this, ChangeActivity.class);
						//						startActivity(intent);
						startActivityForResult(intent, 0);

					}
				});

				popupWindow.showAsDropDown(menuImg, 50, 0);
			}
		});
	}

	public class MFragmentPagerAdapter extends FragmentPagerAdapter{
		public MFragmentPagerAdapter(FragmentManager fm) {
			super(fm);
		}
		@Override
		public Fragment getItem(int arg0) {
			return listFragments.get(arg0);
		}
		@Override
		public int getCount() {
			return listFragments.size();
		}
	}
	
	public void change(int x){
		if(x==0) menuImg.setVisibility(View.INVISIBLE);
		else menuImg.setVisibility(View.VISIBLE);
	}
}
