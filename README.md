# WebViewFragmentSupport
Rewrited WebViewFragment for support v4 fragment

### Code
```java

import android.content.Context;
import android.graphics.Bitmap;
import android.os.Build;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebViewClient;

/**
 * Created by Hivedi2 on 2015-11-09.
 * Rewrited WebViewFragment (http://developer.android.com/reference/android/webkit/WebViewFragment.html)
 * for android.support.v4.app.Fragment
 */
public class WebViewFragmentSupport extends Fragment {

	public static interface OnLoadPage {
		void onPageStart();
		void onPageStop();
	}

	private static final String KEY_URL = "url";
	public static WebViewFragmentSupport newInstance(String url) {
		WebViewFragmentSupport res = new WebViewFragmentSupport();
		Bundle args = new Bundle();
		args.putString(KEY_URL, url);
		res.setArguments(args);
		return res;
	}



	private WebView mWebView;
	private boolean mIsWebViewAvailable;
	private String urlToLoad;
	private OnLoadPage mOnLoad;

	public WebViewFragmentSupport() {
	}

	/**
	 * Called to instantiate the view. Creates and returns the WebView.
	 */
	@Override
	public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
		if (mWebView != null) {
			mWebView.destroy();
		}
		mWebView = new WebView(getActivity());
		mIsWebViewAvailable = true;

		urlToLoad = getArguments() != null ? getArguments().getString(KEY_URL) : "";

		return mWebView;
	}

	@Override
	public void onAttach(Context context) {
		super.onAttach(context);

		mOnLoad = (OnLoadPage) context;
	}

	@Override
	public void onActivityCreated(@Nullable Bundle savedInstanceState) {
		super.onActivityCreated(savedInstanceState);
		WebSettings settings = mWebView.getSettings();

		settings.setJavaScriptEnabled(true);
		settings.setBuiltInZoomControls(true);
		//settings.setUserAgentString(getSettings().getUserAgentString() + Constants.getUserAgent(getContext()));

		if (Build.VERSION.SDK_INT >= 11) {
			settings.setDisplayZoomControls(false);
		}
		if (Build.VERSION.SDK_INT >= 16) {
			settings.setAllowFileAccessFromFileURLs(false);
			settings.setAllowUniversalAccessFromFileURLs(false);
		}
		if (Build.VERSION.SDK_INT >= 17) {
			settings.setMediaPlaybackRequiresUserGesture(true);
		}
		if (Build.VERSION.SDK_INT < 19) { // INFO deprecated in api 18
			//noinspection deprecation
			settings.setPluginState(WebSettings.PluginState.ON);
		}

		settings.setLoadWithOverviewMode(true);
		settings.setUseWideViewPort(true);
		settings.setJavaScriptCanOpenWindowsAutomatically(true);

		// rest
		settings.setDomStorageEnabled(true);
		settings.setAppCachePath(getContext().getCacheDir().toString());
		settings.setAppCacheEnabled(true);
		settings.setCacheMode(WebSettings.LOAD_DEFAULT);
		//settings.setGeolocationDatabasePath(getContext().getCacheDir().getAbsolutePath());
		//settings.setAllowFileAccess(true);
		settings.setDatabaseEnabled(true);
		settings.setSupportZoom(true);
		settings.setBuiltInZoomControls(true);
		//settings.setAllowContentAccess(true);
		settings.setDefaultTextEncodingName("utf-8");

		mWebView.setWebChromeClient(new WebChromeClient());
		mWebView.setWebViewClient(new WebViewClient(){
			@Override
			public void onPageStarted(WebView view, String url, Bitmap favicon) {
				super.onPageStarted(view, url, favicon);
				mOnLoad.onPageStart();
			}

			@Override
			public void onPageFinished(WebView view, String url) {
				super.onPageFinished(view, url);
				mOnLoad.onPageStop();
			}
		});

		mWebView.loadUrl(urlToLoad);
	}

	/**
	 * Called when the fragment is visible to the user and actively running. Resumes the WebView.
	 */
	@Override
	public void onPause() {
		super.onPause();
		mWebView.onPause();
	}

	/**
	 * Called when the fragment is no longer resumed. Pauses the WebView.
	 */
	@Override
	public void onResume() {
		mWebView.onResume();
		super.onResume();
	}

	/**
	 * Called when the WebView has been detached from the fragment.
	 * The WebView is no longer available after this time.
	 */
	@Override
	public void onDestroyView() {
		mIsWebViewAvailable = false;
		super.onDestroyView();
	}

	/**
	 * Called when the fragment is no longer in use. Destroys the internal state of the WebView.
	 */
	@Override
	public void onDestroy() {
		if (mWebView != null) {
			mWebView.destroy();
			mWebView = null;
		}
		super.onDestroy();
	}

	/**
	 * Gets the WebView.
	 */
	@Nullable
	public WebView getWebView() {
		return mIsWebViewAvailable ? mWebView : null;
	}

}

```
