com

ชื่อไฟล์: com/studio/backend/network/NetworkErrorHandler.java

package com.studio.backend.network;

import android.content.Context;
import com.studio.backend.R;

public class NetworkErrorHandler {
	private final Context context;
	
	public NetworkErrorHandler(Context context) {
		this.context = context;
	}
	
	public String getOfflineErrorUrl() {
		return context.getString(R.string.no_connection);
	}
	
	public String getConnectionErrorUrl() {
		return context.getString(R.string.error_connection);
	}
}


ชื่อไฟล์: com/studio/backend/network/NetworkMonitor.java

package com.studio.backend.network;

import android.content.Context;
import android.net.ConnectivityManager;
import android.net.Network;
import android.net.NetworkCapabilities;
import android.net.NetworkRequest;
import android.os.Handler;
import android.os.Looper;

public class NetworkMonitor {
	private final ConnectivityManager connectivityManager;
	private final NetworkStateChecker stateChecker;
	private ConnectivityManager.NetworkCallback networkCallback;
	private final Handler handler = new Handler(Looper.getMainLooper());
	
	public NetworkMonitor(Context context, NetworkStateChecker stateChecker) {
		this.connectivityManager = (ConnectivityManager) context.getSystemService(Context.CONNECTIVITY_SERVICE);
		this.stateChecker = stateChecker;
	}
	
	public void startMonitoring() {
		NetworkRequest request = new NetworkRequest.Builder()
		.addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET)
		.build();
		
		networkCallback = new ConnectivityManager.NetworkCallback() {
			@Override
			public void onAvailable(Network network) {
				handler.post(() -> stateChecker.setNetworkAvailable(true));
			}
			
			@Override
			public void onLost(Network network) {
				handler.post(() -> stateChecker.setNetworkAvailable(false));
			}
		};
		connectivityManager.registerNetworkCallback(request, networkCallback);
	}
	
	public void stopMonitoring() {
		if (networkCallback != null) {
			connectivityManager.unregisterNetworkCallback(networkCallback);
		}
	}
}


ชื่อไฟล์: com/studio/backend/network/NetworkStateChecker.java

package com.studio.backend.network;

import java.util.concurrent.atomic.AtomicBoolean;

public class NetworkStateChecker {
	private static NetworkStateChecker instance;
	private final AtomicBoolean isNetworkAvailable = new AtomicBoolean(false);
	private final NetworkErrorHandler errorHandler;
	
	private NetworkStateChecker(NetworkErrorHandler errorHandler) {
		this.errorHandler = errorHandler;
	}
	
	public static synchronized NetworkStateChecker getInstance(NetworkErrorHandler errorHandler) {
		if (instance == null) {
			instance = new NetworkStateChecker(errorHandler);
		}
		return instance;
	}
	
	public void setNetworkAvailable(boolean available) {
		isNetworkAvailable.set(available);
	}
	
	public String checkNetworkState() {
		return !isNetworkAvailable.get() && errorHandler != null ?
		errorHandler.getOfflineErrorUrl() : null;
	}
	
	public String handleConnectionError() {
		return errorHandler != null ? errorHandler.getConnectionErrorUrl() : null;
	}
	
	public boolean isErrorUrl(String url) {
		if (url == null || errorHandler == null) return false;
		String offlineUrl = errorHandler.getOfflineErrorUrl();
		String errorUrl = errorHandler.getConnectionErrorUrl();
		return url.equals(offlineUrl) || url.equals(errorUrl);
	}
}


ชื่อไฟล์: com/studio/backend/presentation/viewmodel/WebViewViewModel.java

package com.studio.backend.presentation.viewmodel;

import android.app.Application;
import android.content.Context;
import android.content.SharedPreferences;
import androidx.annotation.NonNull;
import androidx.lifecycle.AndroidViewModel;
import androidx.lifecycle.MutableLiveData;

public class WebViewViewModel extends AndroidViewModel {
	private static final String PREFS_NAME = "webview_prefs";
	private static final String KEY_SCROLL_X = "scroll_x";
	private static final String KEY_SCROLL_Y = "scroll_y";
	
	private final MutableLiveData<int[]> scrollPosition = new MutableLiveData<>();
	private final SharedPreferences preferences;
	
	public WebViewViewModel(@NonNull Application application) {
		super(application);
		preferences = application.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
		loadSavedScrollPosition();
	}
	
	public void setScrollPosition(int x, int y) {
		scrollPosition.setValue(new int[]{x, y});
		preferences.edit()
		.putInt(KEY_SCROLL_X, x)
		.putInt(KEY_SCROLL_Y, y)
		.apply();
	}
	
	public int[] getScrollPosition() {
		int[] position = scrollPosition.getValue();
		return position != null ? position : new int[]{0, 0};
	}
	
	private void loadSavedScrollPosition() {
		int x = preferences.getInt(KEY_SCROLL_X, 0);
		int y = preferences.getInt(KEY_SCROLL_Y, 0);
		scrollPosition.setValue(new int[]{x, y});
	}
}


ชื่อไฟล์: com/studio/backend/presentation/viewmodel/WebSitesViewModel.java

package com.studio.backend.presentation.viewmodel;

import androidx.lifecycle.LiveData;
import androidx.lifecycle.MutableLiveData;
import androidx.lifecycle.ViewModel;
import com.studio.backend.domain.model.Site;
import java.util.Arrays;
import java.util.List;

public class WebSitesViewModel extends ViewModel {
	private final MutableLiveData<List<Site>> sites = new MutableLiveData<>();
	private final MutableLiveData<Boolean> isPreloadingComplete = new MutableLiveData<>(false);
	
	public WebSitesViewModel() {
		initializeSites();
	}
	
	private void initializeSites() {
		List<Site> siteList = Arrays.asList(
		new Site("Deepseek", "https://www.deepseek.com/"),
		new Site("Claude", "https://claude.ai/"),
		new Site("Copilot", "https://copilot.microsoft.com/"),
		new Site("Gemini", "https://gemini.google.com/app"),
		new Site("Grok", "https://grok.com/"),
		new Site("ChatGPT", "https://chat.openai.com/"),
		new Site("Hugging Face", "https://huggingface.co/"),
		new Site("Meta AI", "https://www.meta.ai/"),
		new Site("LM Arena", "https://www.lmarena.ai/"),
		new Site("Blackbox", "https://www.blackbox.ai/"),
		new Site("Phind", "https://www.phind.com/"),
		new Site("You AI", "https://you.com/"),
		new Site("Mistral AI", "https://chat.mistral.ai/chat"),
		new Site("Cohere", "https://cohere.ai/"),
		new Site("Groq", "https://groq.com/"),
		new Site("DeepInfra", "https://deepinfra.com/")
		);
		sites.setValue(siteList);
	}
	
	public LiveData<List<Site>> getSites() {
		return sites;
	}
	
	public LiveData<Boolean> getPreloadingStatus() {
		return isPreloadingComplete;
	}
	
	public void setPreloadingComplete(boolean complete) {
		isPreloadingComplete.setValue(complete);
	}
}


ชื่อไฟล์: com/studio/backend/presentation/fragments/WebsiteSelectionFragment.java

package com.studio.backend.presentation.fragments;

public class WebsiteSelectionFragment {
	
}


ชื่อไฟล์: com/studio/backend/presentation/activity/WebViewActivity.java

package com.studio.backend.presentation.activity;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.view.ViewGroup;
import android.webkit.WebView;
import android.widget.ImageView;
import android.widget.ProgressBar;
import android.widget.RelativeLayout;
import androidx.appcompat.app.AppCompatActivity;
import androidx.lifecycle.ViewModelProvider;
import com.studio.backend.R;
import com.studio.backend.management.resource.WebRenderOptimizer;
import com.studio.backend.management.resource.WebResourceCleaner;
import com.studio.backend.management.resource.WebResourceOptimizer;
import com.studio.backend.management.scripts.JavaScriptExecutor;
import com.studio.backend.management.scripts.JavaScriptInterface;
import com.studio.backend.management.webview.WebChromeClientManager;
import com.studio.backend.management.webview.WebViewClientManager;
import com.studio.backend.management.webview.WebViewSettingManager;
import com.studio.backend.management.webview.WebViewUploadManager;
import com.studio.backend.network.NetworkErrorHandler;
import com.studio.backend.network.NetworkMonitor;
import com.studio.backend.network.NetworkStateChecker;
import com.studio.backend.presentation.viewmodel.WebViewViewModel;
import com.studio.backend.storage.cache.WebCacheManager;
import com.studio.backend.storage.cookie.WebCookieManager;
import com.studio.backend.storage.prefs.LinkPreferencesManager;

public class WebViewActivity extends AppCompatActivity {
	private WebView webView;
	private ProgressBar progressBar;
	private ImageView backButton;
	private ImageView reloadButton;
	private View homeIndicator;
	private RelativeLayout headerLayout;
	private WebViewViewModel viewModel;
	private LinkPreferencesManager linkPreferencesManager;
	private WebResourceOptimizer resourceOptimizer;
	private WebRenderOptimizer renderOptimizer;
	private WebViewUploadManager uploadManager;
	private JavaScriptExecutor jsExecutor;
	private WebCacheManager cacheManager;
	private NetworkMonitor networkMonitor;
	private WebResourceCleaner resourceCleaner;
	private WebCookieManager cookieManager;
	private NetworkStateChecker networkStateChecker;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_webview);
		initializeComponents();
		setupWebView();
		setupEventListeners();
		loadInitialContent(savedInstanceState);
	}
	
	private void initializeComponents() {
		webView = findViewById(R.id.webview);
		progressBar = findViewById(R.id.progress_bar);
		backButton = findViewById(R.id.back_button);
		reloadButton = findViewById(R.id.reload_button);
		homeIndicator = findViewById(R.id.home_indicator);
		headerLayout = findViewById(R.id.header_layout);
		viewModel = new ViewModelProvider(this).get(WebViewViewModel.class);
		linkPreferencesManager = new LinkPreferencesManager(this);
		uploadManager = new WebViewUploadManager(this);
		jsExecutor = new JavaScriptExecutor(webView, this);
		resourceOptimizer = new WebResourceOptimizer(webView);
		renderOptimizer = new WebRenderOptimizer(webView);
		cacheManager = new WebCacheManager(this);
		cookieManager = new WebCookieManager(this);
		NetworkErrorHandler errorHandler = new NetworkErrorHandler(this);
		networkStateChecker = NetworkStateChecker.getInstance(errorHandler);
		resourceCleaner = new WebResourceCleaner(webView, cookieManager);
		networkMonitor = new NetworkMonitor(this, networkStateChecker);
	}
	
	private void setupWebView() {
		WebViewSettingManager.applySettings(webView);
		webView.addJavascriptInterface(new JavaScriptInterface(this, webView, viewModel, headerLayout), "AndroidInterface");
		webView.setWebViewClient(new WebViewClientManager(
		this, progressBar, backButton, homeIndicator, viewModel, linkPreferencesManager,
		this::updateBackButtonState, jsExecutor, networkStateChecker));
		webView.setWebChromeClient(new WebChromeClientManager(progressBar, uploadManager));
		renderOptimizer.enableTurboMode();
		jsExecutor.executeCriticalScriptsFirst();
	}
	
	private void setupEventListeners() {
		backButton.setOnClickListener(v -> navigateBack());
		reloadButton.setOnClickListener(v -> reloadDefaultPage());
		homeIndicator.setOnClickListener(v -> reloadDefaultPage());
		updateBackButtonState();
	}
	
	private void loadInitialContent(Bundle savedInstanceState) {
		if (savedInstanceState != null) {
			webView.restoreState(savedInstanceState);
			} else {
			String lastUrl = linkPreferencesManager.getLastUrl();
			if (lastUrl != null && !lastUrl.isEmpty()) {
				loadPageFromCacheOrNetwork(lastUrl);
				} else {
				loadPageFromCacheOrNetwork(getString(R.string.url_select));
			}
		}
	}
	
	private void loadPageFromCacheOrNetwork(String url) {
		cacheManager.loadHtml(url, new WebCacheManager.Callback() {
			@Override
			public void onResult(String html) {
				runOnUiThread(() -> webView.loadDataWithBaseURL(url, html, "text/html", "UTF-8", null));
			}
			
			@Override
			public void onError(Exception e) {
				runOnUiThread(() -> webView.loadUrl(url));
			}
		});
	}
	
	private void reloadDefaultPage() {
		loadPageFromCacheOrNetwork(getString(R.string.url_select));
	}
	
	private void navigateBack() {
		if (webView.canGoBack()) {
			webView.goBack();
		}
	}
	
	@Override
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		super.onActivityResult(requestCode, resultCode, data);
		uploadManager.handleActivityResult(requestCode, resultCode, data);
	}
	
	@Override
	protected void onResume() {
		super.onResume();
		resourceOptimizer.onResume();
		renderOptimizer.onResume();
		networkMonitor.startMonitoring();
		restoreScrollPosition();
	}
	
	private void restoreScrollPosition() {
		webView.postDelayed(() -> {
			int[] scrollPosition = viewModel.getScrollPosition();
			webView.scrollTo(scrollPosition[0], scrollPosition[1]);
		}, 300);
	}
	
	@Override
	protected void onPause() {
		super.onPause();
		resourceOptimizer.onPause();
		renderOptimizer.onPause();
		networkMonitor.stopMonitoring();
		saveScrollPosition();
		saveCurrentUrl();
	}
	
	private void saveScrollPosition() {
		viewModel.setScrollPosition(webView.getScrollX(), webView.getScrollY());
	}
	
	private void saveCurrentUrl() {
		String currentUrl = webView.getUrl();
		if (currentUrl != null && !currentUrl.isEmpty() && !currentUrl.startsWith("about:blank")) {
			linkPreferencesManager.saveLastUrl(currentUrl);
		}
	}
	
	@Override
	protected void onSaveInstanceState(Bundle outState) {
		super.onSaveInstanceState(outState);
		webView.saveState(outState);
	}
	
	@Override
	protected void onDestroy() {
		saveCurrentUrl();
		networkMonitor.stopMonitoring();
		cleanupResources();
		super.onDestroy();
	}
	
	private void cleanupResources() {
		resourceCleaner.cleanup();
		renderOptimizer.onDestroy();
		if (webView != null) {
			((ViewGroup) webView.getParent()).removeView(webView);
			webView.destroy();
		}
	}
	
	private void updateBackButtonState() {
		String currentUrl = webView.getUrl();
		String urlSelect = getString(R.string.url_select);
		backButton.setAlpha(currentUrl != null && currentUrl.equals(urlSelect) ? 0.3f : 1.0f);
		backButton.setEnabled(webView.canGoBack());
	}
	
	@Override
	public void onBackPressed() {
		if (webView.canGoBack()) {
			webView.goBack();
			} else {
			super.onBackPressed();
		}
	}
}


ชื่อไฟล์: com/studio/backend/management/resource/WebRenderOptimizer.java

package com.studio.backend.management.resource;

import android.os.Build;
import android.view.View;
import android.webkit.WebSettings;
import android.webkit.WebView;

public class WebRenderOptimizer {
	private final WebView webView;
	
	public WebRenderOptimizer(WebView webView) {
		if (webView == null) {
			throw new IllegalArgumentException("WebView cannot be null");
		}
		this.webView = webView;
		configureInitialRenderingSettings();
	}
	
	private void configureInitialRenderingSettings() {
		WebSettings settings = webView.getSettings();
		settings.setEnableSmoothTransition(true);
		webView.setLayerType(View.LAYER_TYPE_HARDWARE, null);
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
			settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.TEXT_AUTOSIZING);
			} else {
			settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.NORMAL);
		}
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
			settings.setOffscreenPreRaster(true);
		}
	}
	
	public void enableTurboMode() {
		webView.setLayerType(View.LAYER_TYPE_HARDWARE, null);
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
			webView.getSettings().setOffscreenPreRaster(true);
		}
	}
	
	public void onResume() {
		webView.onResume();
		WebSettings settings = webView.getSettings();
		settings.setLoadsImagesAutomatically(true);
		settings.setBlockNetworkImage(false);
		if (webView.getLayerType() != View.LAYER_TYPE_HARDWARE) {
			webView.setLayerType(View.LAYER_TYPE_HARDWARE, null);
		}
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
			settings.setOffscreenPreRaster(true);
		}
	}
	
	public void onPause() {
		webView.onPause();
		if (webView.getLayerType() == View.LAYER_TYPE_HARDWARE) {
			webView.setLayerType(View.LAYER_TYPE_NONE, null);
		}
	}
	
	public void onDestroy() {
	}
}


ชื่อไฟล์: com/studio/backend/management/resource/WebResourceCleaner.java

package com.studio.backend.management.resource;

import android.webkit.CookieManager;
import android.webkit.WebView;
import com.studio.backend.storage.cookie.WebCookieManager;
import java.lang.ref.WeakReference;

public class WebResourceCleaner {
    private final WeakReference<WebView> webViewRef;
    private final WebCookieManager cookieManager;

    public WebResourceCleaner(WebView webView, WebCookieManager cookieManager) {
        this.webViewRef = new WeakReference<>(webView);
        this.cookieManager = cookieManager;
    }

    public void cleanup() {
        WebView webView = webViewRef.get();
        if (webView != null) {
            webView.stopLoading();
            webView.setWebChromeClient(null);
            webView.setWebViewClient(null);
            webView.clearHistory();
            webView.clearCache(false);
            webView.loadUrl("about:blank");
            webView.freeMemory();
            webView.pauseTimers();
            CookieManager.getInstance().removeAllCookies(null);
        }
        webViewRef.clear();
    }
}


ชื่อไฟล์: com/studio/backend/management/resource/WebViewPreloader.java

package com.studio.backend.management.resource;

import android.content.Context;
import com.studio.backend.domain.model.Site;
import com.studio.backend.storage.cache.WebCacheManager;
import java.util.List;

public class WebViewPreloader {
	private final WebCacheManager cacheManager;
	private final List<Site> sites;
	
	public WebViewPreloader(Context context, List<Site> sites) {
		this.cacheManager = new WebCacheManager(context);
		this.sites = sites;
	}
	
	public void preloadSites() {
		for (Site site : sites) {
			cacheManager.loadHtml(site.getUrl(), new WebCacheManager.Callback() {
				@Override
				public void onResult(String html) {}
				
				@Override
				public void onError(Exception e) {}
			});
		}
	}
}


ชื่อไฟล์: com/studio/backend/management/resource/WebResourceOptimizer.java

package com.studio.backend.management.resource;

import android.webkit.WebView;
import java.lang.ref.WeakReference;

public class WebResourceOptimizer {
	private final WeakReference<WebView> webViewRef;
	
	public WebResourceOptimizer(WebView webView) {
		this.webViewRef = new WeakReference<>(webView);
	}
	
	public void onPause() {
		WebView webView = webViewRef.get();
		if (webView != null) {
			webView.onPause();
			webView.pauseTimers();
			webView.evaluateJavascript("if(typeof document!== 'undefined' && typeof Event!== 'undefined'){ document.dispatchEvent(new Event('appPause')); }", null);
		}
	}
	
	public void onResume() {
		WebView webView = webViewRef.get();
		if (webView != null) {
			webView.onResume();
			webView.resumeTimers();
			webView.evaluateJavascript("if(typeof window!== 'undefined' && typeof window.resumeAllMedia === 'function'){ window.resumeAllMedia(); }", null);
		}
	}
}


ชื่อไฟล์: com/studio/backend/management/webview/WebChromeClientManager.java

package com.studio.backend.management.webview;

import android.net.Uri;
import android.view.View;
import android.webkit.ValueCallback;
import android.webkit.WebChromeClient;
import android.webkit.WebView;
import android.widget.ProgressBar;

public class WebChromeClientManager extends WebChromeClient {
	
	private final ProgressBar progressBar;
	private final WebViewUploadManager uploadManager;
	
	public WebChromeClientManager(ProgressBar progressBar, WebViewUploadManager uploadManager) {
		this.progressBar = progressBar;
		this.uploadManager = uploadManager;
	}
	
	@Override
	public void onProgressChanged(WebView view, int newProgress) {
		super.onProgressChanged(view, newProgress);
		
		if (progressBar != null) {
			progressBar.setProgress(newProgress);
			
			if (newProgress < 100 && progressBar.getVisibility() == View.GONE) {
				progressBar.setVisibility(View.VISIBLE);
			}
			
			if (newProgress == 100) {
				progressBar.postDelayed(() -> progressBar.setVisibility(View.GONE), 150);
			}
		}
	}
	
	@Override
	public boolean onShowFileChooser(WebView webView,
	ValueCallback<Uri[]> filePathCallback,
	FileChooserParams fileChooserParams) {
		if (uploadManager != null) {
			return uploadManager.onShowFileChooser(webView, filePathCallback, fileChooserParams);
		}
		return false;
	}
}


ชื่อไฟล์: com/studio/backend/management/webview/WebViewClientManager.java

package com.studio.backend.management.webview;

import android.content.Context;
import android.graphics.Bitmap;
import android.os.Build;
import android.view.View;
import android.webkit.WebResourceError;
import android.webkit.WebResourceRequest;
import android.webkit.WebView;
import android.webkit.WebViewClient;
import android.widget.ImageView;
import android.widget.ProgressBar;
import androidx.annotation.NonNull;
import com.studio.backend.network.NetworkStateChecker;
import com.studio.backend.storage.prefs.LinkPreferencesManager;
import com.studio.backend.management.scripts.JavaScriptExecutor;
import com.studio.backend.presentation.viewmodel.WebViewViewModel;

public class WebViewClientManager extends WebViewClient {
	private final ProgressBar progressBar;
	private final LinkPreferencesManager linkPreferencesManager;
	private final Runnable updateBackButtonState;
	private final JavaScriptExecutor javaScriptExecutor;
	private final NetworkStateChecker networkStateChecker;
	
	public WebViewClientManager(
	@NonNull Context context,
	@NonNull ProgressBar progressBar,
	@NonNull ImageView backButton,
	@NonNull View homeIndicator,
	@NonNull WebViewViewModel viewModel,
	@NonNull LinkPreferencesManager linkPreferencesManager,
	@NonNull Runnable updateBackButtonState,
	@NonNull JavaScriptExecutor javaScriptExecutor,
	@NonNull NetworkStateChecker networkStateChecker) {
		this.progressBar = progressBar;
		this.linkPreferencesManager = linkPreferencesManager;
		this.updateBackButtonState = updateBackButtonState;
		this.javaScriptExecutor = javaScriptExecutor;
		this.networkStateChecker = networkStateChecker;
	}
	
	@Override
	public void onPageStarted(WebView view, String url, Bitmap favicon) {
		progressBar.setVisibility(View.VISIBLE);
		progressBar.setProgress(0);
		updateBackButtonState.run();
		
		String offlineErrorUrl = networkStateChecker.checkNetworkState();
		if (offlineErrorUrl != null && !networkStateChecker.isErrorUrl(url)) {
			view.stopLoading();
			view.loadUrl(offlineErrorUrl);
		}
	}
	
	@Override
	public void onPageFinished(WebView view, String url) {
		if (url != null && !url.isEmpty() && !networkStateChecker.isErrorUrl(url) &&
		!url.startsWith("about:blank") && !url.startsWith("file:///android_asset/") &&
		!url.startsWith("javascript:")) {
			linkPreferencesManager.saveLastUrl(url);
		}
		updateBackButtonState.run();
		if (!networkStateChecker.isErrorUrl(url)) {
			javaScriptExecutor.executeCriticalScriptsFirst();
		}
		progressBar.setVisibility(View.GONE);
	}
	
	@Override
	public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
		handleError(view, request.getUrl().toString());
	}
	
	@Override
	public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
		handleError(view, failingUrl);
	}
	
	private void handleError(WebView view, String failingUrl) {
		progressBar.setVisibility(View.GONE);
		if (!networkStateChecker.isErrorUrl(failingUrl)) {
			String errorUrl = networkStateChecker.handleConnectionError();
			if (errorUrl != null && !errorUrl.equals(view.getUrl())) {
				view.loadUrl(errorUrl);
			}
		}
	}
}


ชื่อไฟล์: com/studio/backend/management/webview/WebViewUploadManager.java

package com.studio.backend.management.webview;

import android.app.Activity;
import android.content.Intent;
import android.net.Uri;
import android.webkit.ValueCallback;
import android.webkit.WebChromeClient;
import android.webkit.WebView;
import android.widget.Toast;

public class WebViewUploadManager {
	private static final int FILE_CHOOSER_REQUEST_CODE = 1;
	private final Activity activity;
	private ValueCallback<Uri[]> filePathCallback;
	
	public WebViewUploadManager(Activity activity) {
		this.activity = activity;
	}
	
	public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback,
	WebChromeClient.FileChooserParams fileChooserParams) {
		if (this.filePathCallback != null) {
			this.filePathCallback.onReceiveValue(null);
		}
		this.filePathCallback = filePathCallback;
		
		Intent intent = fileChooserParams.createIntent();
		try {
			activity.startActivityForResult(intent, FILE_CHOOSER_REQUEST_CODE);
			} catch (Exception e) {
			Toast.makeText(activity, "Cannot open file chooser", Toast.LENGTH_LONG).show();
			return false;
		}
		
		return true;
	}
	
	public boolean handleActivityResult(int requestCode, int resultCode, Intent data) {
		if (requestCode != FILE_CHOOSER_REQUEST_CODE || filePathCallback == null) {
			return false;
		}
		
		Uri[] results = null;
		if (resultCode == Activity.RESULT_OK) {
			if (data != null) {
				String dataString = data.getDataString();
				if (dataString != null) {
					results = new Uri[]{Uri.parse(dataString)};
					} else if (data.getClipData() != null) {
					int count = data.getClipData().getItemCount();
					results = new Uri[count];
					for (int i = 0; i < count; i++) {
						results[i] = data.getClipData().getItemAt(i).getUri();
					}
				}
			}
		}
		
		filePathCallback.onReceiveValue(results);
		filePathCallback = null;
		return true;
	}
}


ชื่อไฟล์: com/studio/backend/management/webview/WebViewSettingManager.java

package com.studio.backend.management.webview;

import android.os.Build;
import android.webkit.CookieManager;
import android.webkit.WebSettings;
import android.webkit.WebView;
import com.studio.backend.R;

public class WebViewSettingManager {
	public static void applySettings(WebView webView) {
		WebSettings webSettings = webView.getSettings();
		configureCoreSettings(webView, webSettings);
		configureJavaScriptSettings(webSettings);
		configureFileAccessSettings(webSettings);
		configureMediaSettings(webSettings);
		configureCacheSettings(webView, webSettings);
		configureMixedContentSettings(webSettings);
		configureDarkModeSettings(webSettings);
		configureCookieSettings(webView);
	}
	
	public static void configureCoreSettings(WebView webView, WebSettings webSettings) {
		webSettings.setUserAgentString(webView.getContext().getString(R.string.user_agent));
		webSettings.setLoadWithOverviewMode(true);
		webSettings.setUseWideViewPort(true);
		webSettings.setBuiltInZoomControls(true);
		webSettings.setDisplayZoomControls(false);
		webSettings.setGeolocationEnabled(true);
		webSettings.setSupportMultipleWindows(true);
		webSettings.setJavaScriptCanOpenWindowsAutomatically(true);
	}
	
	public static void configureJavaScriptSettings(WebSettings webSettings) {
		webSettings.setJavaScriptEnabled(true);
		webSettings.setDomStorageEnabled(true);
		webSettings.setDatabaseEnabled(true);
	}
	
	public static void configureFileAccessSettings(WebSettings webSettings) {
		webSettings.setAllowFileAccess(true);
		webSettings.setAllowContentAccess(true);
		webSettings.setAllowUniversalAccessFromFileURLs(true);
		webSettings.setAllowFileAccessFromFileURLs(true);
	}
	
	public static void configureMediaSettings(WebSettings webSettings) {
		webSettings.setLoadsImagesAutomatically(true);
		webSettings.setBlockNetworkImage(false);
		webSettings.setBlockNetworkLoads(false);
		webSettings.setMediaPlaybackRequiresUserGesture(false);
	}
	
	public static void configureCacheSettings(WebView webView, WebSettings webSettings) {
		webSettings.setCacheMode(WebSettings.LOAD_DEFAULT);
		webSettings.setAppCacheEnabled(true);
		webSettings.setAppCachePath(webView.getContext().getCacheDir().getAbsolutePath());
	}
	
	public static void configureMixedContentSettings(WebSettings webSettings) {
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
			webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
			} else {
			webSettings.setMixedContentMode(WebSettings.MIXED_CONTENT_COMPATIBILITY_MODE);
		}
	}
	
	public static void configureDarkModeSettings(WebSettings webSettings) {
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
			webSettings.setForceDark(WebSettings.FORCE_DARK_AUTO);
		}
	}
	
	public static void configureCookieSettings(WebView webView) {
		CookieManager cookieManager = CookieManager.getInstance();
		cookieManager.setAcceptCookie(true);
		cookieManager.setAcceptThirdPartyCookies(webView, true);
	}
	
	public static void setDarkMode(WebView webView, boolean darkModeEnabled) {
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
			webView.getSettings().setForceDark(
			darkModeEnabled ? WebSettings.FORCE_DARK_ON : WebSettings.FORCE_DARK_OFF);
		}
	}
	
	public static void enableTurboMode(WebView webView) {
		WebSettings webSettings = webView.getSettings();
		webSettings.setMediaPlaybackRequiresUserGesture(false);
		webSettings.setJavaScriptCanOpenWindowsAutomatically(true);
		webSettings.setSupportMultipleWindows(true);
		webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);
		webSettings.setLoadsImagesAutomatically(true);
		webSettings.setBlockNetworkImage(false);
	}
}


ชื่อไฟล์: com/studio/backend/management/scripts/JavaScriptInterface.java

package com.studio.backend.management.scripts;

import android.content.Context;
import android.graphics.Color;
import android.graphics.drawable.GradientDrawable;
import android.view.View;
import android.webkit.JavascriptInterface;
import android.webkit.WebView;
import androidx.annotation.NonNull;
import com.studio.backend.R;
import com.studio.backend.storage.prefs.LinkPreferencesManager;
import com.studio.backend.presentation.viewmodel.WebViewViewModel;

public class JavaScriptInterface {
	private final Context context;
	private final WebView webView;
	private final WebViewViewModel viewModel;
	private final LinkPreferencesManager preferencesManager;
	private final View headerLayout;
	
	public JavaScriptInterface(
	@NonNull Context context,
	@NonNull WebView webView,
	@NonNull WebViewViewModel viewModel,
	@NonNull View headerLayout
	) {
		this.context = context;
		this.webView = webView;
		this.viewModel = viewModel;
		this.preferencesManager = new LinkPreferencesManager(context);
		this.headerLayout = headerLayout;
	}
	
	@JavascriptInterface
	public void updateWebsiteUrl(String url) {
		if (url != null && !url.trim().isEmpty()) {
			preferencesManager.saveLastUrl(url.trim());
			loadUrlOnUiThread(url.trim());
		}
	}
	
	@JavascriptInterface
	public void saveScrollPosition(int x, int y) {
		viewModel.setScrollPosition(x, y);
	}
	
	@JavascriptInterface
	public void reloadPage() {
		String url = preferencesManager.getLastUrl();
		if (url != null && !url.isEmpty()) {
			loadUrlOnUiThread(url);
		}
	}
	
	@JavascriptInterface
	public void loadErrorPage(String errorPageUrl) {
		loadUrlOnUiThread(errorPageUrl);
	}
	
	@JavascriptInterface
	public void setDynamicBackgroundColor(String hexColor) {
		if (hexColor != null && hexColor.startsWith("#")) {
			try {
				headerLayout.post(() -> {
					GradientDrawable newDrawable = new GradientDrawable();
					newDrawable.setColor(Color.parseColor(hexColor));
					newDrawable.setCornerRadii(new float[] {
						25f, 25f,
						25f, 25f,
						0f, 0f,
						0f, 0f
					});
					headerLayout.setBackground(newDrawable);
				});
				} catch (IllegalArgumentException e) {
				headerLayout.post(() -> {
					headerLayout.setBackgroundResource(R.drawable.header_background);
				});
			}
		}
	}
	
	private void loadUrlOnUiThread(String url) {
		if (webView != null) {
			webView.post(() -> webView.loadUrl(url));
		}
	}
}


ชื่อไฟล์: com/studio/backend/management/scripts/JavaScriptExecutor.java

package com.studio.backend.management.scripts;

import android.content.Context;
import android.content.res.AssetManager;
import android.os.Handler;
import android.os.Looper;
import android.webkit.WebView;
import androidx.annotation.NonNull;
import androidx.annotation.WorkerThread;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class JavaScriptExecutor {
	private static final String BOOKMARK_PATH = "bookmark/";
	private static final long DELAY_BETWEEN_SCRIPTS_MS = 2000;
	
	private final WebView webView;
	private final AssetManager assetManager;
	private final Map<String, String> scriptCache = new ConcurrentHashMap<>();
	private final Handler mainHandler = new Handler(Looper.getMainLooper());
	
	public JavaScriptExecutor(@NonNull WebView webView, @NonNull Context context) {
		this.webView = webView;
		this.assetManager = context.getAssets();
	}
	
	public void executeCriticalScriptsFirst() {
		new Thread(this::loadAndExecuteScripts).start();
	}
	
	@WorkerThread
	private void loadAndExecuteScripts() {
		List<String> scripts = loadScriptsFromAssets();
		if (scripts.isEmpty()) {
			return;
		}
		
		executeScriptOnMainThread(scripts.get(0));
		
		if (scripts.size() > 1) {
			mainHandler.postDelayed(() -> {
				for (int i = 1; i < scripts.size(); i++) {
					executeScriptOnMainThread(scripts.get(i));
				}
			}, DELAY_BETWEEN_SCRIPTS_MS);
		}
	}
	
	private List<String> loadScriptsFromAssets() {
		try {
			String[] files = assetManager.list(BOOKMARK_PATH);
			if (files == null || files.length == 0) {
				return Collections.emptyList();
			}
			
			List<String> scripts = new ArrayList<>(files.length);
			for (String file : files) {
				if (file.endsWith(".js")) {
					String script = getCachedScript(file);
					if (script != null) {
						scripts.add(script);
					}
				}
			}
			return scripts;
			} catch (IOException e) {
			return Collections.emptyList();
		}
	}
	
	private String getCachedScript(String fileName) {
		return scriptCache.computeIfAbsent(fileName, key -> {
			try {
				return readScriptFromAsset(BOOKMARK_PATH + key);
				} catch (IOException e) {
				return null;
			}
		});
	}
	
	private String readScriptFromAsset(String path) throws IOException {
		StringBuilder script = new StringBuilder();
		try (InputStream inputStream = assetManager.open(path);
		BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream))) {
			String line;
			while ((line = reader.readLine()) != null) {
				script.append(line).append("\n");
			}
			return script.toString();
		}
	}
	
	private void executeScriptOnMainThread(String script) {
		if (script == null || script.trim().isEmpty()) {
			return;
		}
		mainHandler.post(() -> {
			if (webView != null) {
				webView.evaluateJavascript(script, null);
			}
		});
	}
}


ชื่อไฟล์: com/studio/backend/MyApplication.java

package com.studio.backend;

import android.app.Application;
import android.content.Context;
import androidx.multidex.MultiDex;

public class MyApplication extends Application {
	private static MyApplication instance;
	
	@Override
	public void onCreate() {
		super.onCreate();
		instance = this;
	}
	
	@Override
	protected void attachBaseContext(Context base) {
		super.attachBaseContext(base);
		MultiDex.install(this);
	}
	
	public static MyApplication getInstance() {
		return instance;
	}
	
	public static Context getAppContext() {
		return instance.getApplicationContext();
	}
}


ชื่อไฟล์: com/studio/backend/storage/cache/WebCacheManager.java

package com.studio.backend.storage.cache;

import android.content.Context;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
import com.studio.backend.network.NetworkErrorHandler;
import com.studio.backend.network.NetworkStateChecker;
import java.io.IOException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

public class WebCacheManager {
	private final WebCacheRepository repository;
	private final OkHttpClient httpClient;
	private final ExecutorService executor;
	private final Context context;
	private final NetworkStateChecker networkStateChecker;
	private final NetworkErrorHandler errorHandler;
	
	public WebCacheManager(Context context) {
		this.context = context;
		this.repository = new WebCacheRepository(context);
		this.errorHandler = new NetworkErrorHandler(context);
		this.networkStateChecker = NetworkStateChecker.getInstance(errorHandler);
		
		this.httpClient = new OkHttpClient.Builder()
		.connectTimeout(15, java.util.concurrent.TimeUnit.SECONDS)
		.readTimeout(15, java.util.concurrent.TimeUnit.SECONDS)
		.build();
		
		this.executor = Executors.newFixedThreadPool(4);
	}
	
	public interface Callback {
		void onResult(String html);
		void onError(Exception e);
	}
	
	public void loadHtml(String url, Callback callback) {
		executor.submit(() -> {
			try {
				String cached = repository.get(url);
				if (cached != null) {
					callback.onResult(cached);
					return;
				}
				
				String networkError = networkStateChecker.checkNetworkState();
				if (networkError != null) {
					callback.onResult(networkError);
					return;
				}
				
				Request request = new Request.Builder()
				.url(url)
				.header("Cache-Control", "no-cache")
				.build();
				
				try (Response response = httpClient.newCall(request).execute()) {
					if (response.isSuccessful() && response.body() != null) {
						String html = response.body().string();
						repository.save(url, html);
						callback.onResult(html);
						} else {
						handleFailedResponse(callback);
					}
				}
				} catch (IOException e) {
				handleNetworkError(callback);
				} catch (Exception e) {
				callback.onError(e);
			}
		});
	}
	
	private void handleFailedResponse(Callback callback) {
		String errorPage = networkStateChecker.handleConnectionError();
		if (errorPage != null) {
			callback.onResult(errorPage);
			} else {
			callback.onError(new IOException("Request failed"));
		}
	}
	
	private void handleNetworkError(Callback callback) {
		String errorPage = networkStateChecker.handleConnectionError();
		if (errorPage != null) {
			callback.onResult(errorPage);
			} else {
			callback.onResult(getOfflineFallbackHtml());
		}
	}
	
	private String getOfflineFallbackHtml() {
		return errorHandler.getOfflineErrorUrl();
	}
	
	public void shutdown() {
		executor.shutdown();
		repository.close();
	}
}


ชื่อไฟล์: com/studio/backend/storage/cache/WebCacheEntity.java

package com.studio.backend.storage.cache;

import java.util.Date;

public class WebCacheEntity {
	public String url;
	public String html;
	public long timestamp;
	
	public WebCacheEntity(String url, String html) {
		this.url = url;
		this.html = html;
		this.timestamp = new Date().getTime();
	}
}


ชื่อไฟล์: com/studio/backend/storage/cache/WebCacheRepository.java

package com.studio.backend.storage.cache;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;
import java.util.Date;

public class WebCacheRepository {
	private static final String DATABASE_NAME = "web_cache.db";
	private static final int DATABASE_VERSION = 2;
	private static final long MAX_DB_SIZE = 10 * 1024 * 1024;
	private static final long CACHE_EXPIRATION_MS = 3 * 24 * 60 * 60 * 1000;
	
	private final DatabaseHelper dbHelper;
	
	private static class DatabaseHelper extends SQLiteOpenHelper {
		DatabaseHelper(Context context) {
			super(context, DATABASE_NAME, null, DATABASE_VERSION);
		}
		
		@Override
		public void onCreate(SQLiteDatabase db) {
			db.execSQL("CREATE TABLE web_cache (url TEXT PRIMARY KEY, html TEXT, timestamp INTEGER)");
		}
		
		@Override
		public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
			if (oldVersion < 2) {
				db.execSQL("ALTER TABLE web_cache ADD COLUMN timestamp INTEGER");
			}
		}
	}
	
	public WebCacheRepository(Context context) {
		dbHelper = new DatabaseHelper(context);
	}
	
	public synchronized void save(String url, String html) {
		SQLiteDatabase db = dbHelper.getWritableDatabase();
		try {
			ContentValues values = new ContentValues();
			values.put("url", url);
			values.put("html", html);
			values.put("timestamp", new Date().getTime());
			db.insertWithOnConflict("web_cache", null, values, SQLiteDatabase.CONFLICT_REPLACE);
			maintainDbSize(db);
			} finally {
			db.close();
		}
	}
	
	public synchronized String get(String url) {
		SQLiteDatabase db = dbHelper.getReadableDatabase();
		Cursor cursor = null;
		try {
			cursor = db.rawQuery(
			"SELECT html FROM web_cache WHERE url = ? AND timestamp > ?",
			new String[]{url, String.valueOf(new Date().getTime() - CACHE_EXPIRATION_MS)}
			);
			return cursor.moveToFirst() ? cursor.getString(0) : null;
			} finally {
			if (cursor != null) cursor.close();
			db.close();
		}
	}
	
	private void maintainDbSize(SQLiteDatabase db) {
		Cursor cursor = null;
		try {
			cursor = db.rawQuery("SELECT SUM(LENGTH(html)) FROM web_cache", null);
			if (cursor.moveToFirst() && cursor.getLong(0) > MAX_DB_SIZE) {
				db.execSQL("DELETE FROM web_cache WHERE url IN " +
				"(SELECT url FROM web_cache ORDER BY timestamp ASC LIMIT 10)");
			}
			} finally {
			if (cursor != null) cursor.close();
		}
	}
	
	public synchronized void clearExpired() {
		SQLiteDatabase db = dbHelper.getWritableDatabase();
		try {
			db.execSQL("DELETE FROM web_cache WHERE timestamp <= ?",
			new String[]{String.valueOf(new Date().getTime() - CACHE_EXPIRATION_MS)});
			} finally {
			db.close();
		}
	}
	
	public void close() {
		dbHelper.close();
	}
}


ชื่อไฟล์: com/studio/backend/storage/prefs/LinkPreferencesManager.java

package com.studio.backend.storage.prefs;

import android.content.Context;
import android.content.SharedPreferences;
import androidx.annotation.NonNull;

public class LinkPreferencesManager {
	private static final String PREFS_NAME = "link_prefs";
	private static final String KEY_LAST_URL = "last_url";
	private final SharedPreferences preferences;
	
	public LinkPreferencesManager(@NonNull Context context) {
		preferences = context.getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);
	}
	
	public void saveLastUrl(String url) {
		if (url != null && url.startsWith("https")) {
			preferences.edit().putString(KEY_LAST_URL, url).apply();
		}
	}
	
	public String getLastUrl() {
		return preferences.getString(KEY_LAST_URL, null);
	}
}


ชื่อไฟล์: com/studio/backend/storage/cookie/WebCookieManager.java

package com.studio.backend.storage.cookie;

import android.content.Context;
import android.webkit.CookieManager;
import androidx.annotation.NonNull;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

public class WebCookieManager {
	private final CookieManager cookieManager;
	
	public WebCookieManager(@NonNull Context context) {
		this.cookieManager = CookieManager.getInstance();
	}
	
	public void clearExpiredCookies() {
		long currentTime = System.currentTimeMillis();
		long threeDaysInMillis = 3 * 24 * 60 * 60 * 1000;
		String cookies = cookieManager.getCookie("https://*");
		if (cookies != null) {
			String[] cookieArray = cookies.split(";");
			SimpleDateFormat dateFormat = new SimpleDateFormat("EEE, dd-MMM-yyyy HH:mm:ss z", Locale.US);
			for (String cookie : cookieArray) {
				String[] parts = cookie.split("=");
				if (parts.length > 0) {
					String name = parts[0].trim();
					String[] attrs = cookie.split(";");
					long expiresAt = currentTime;
					for (String attr : attrs) {
						if (attr.trim().startsWith("Expires=")) {
							try {
								String dateStr = attr.split("=", 2)[1].trim();
								Date expiryDate = dateFormat.parse(dateStr);
								if (expiryDate != null) {
									expiresAt = expiryDate.getTime();
								}
							} catch (Exception ignored) {}
						}
					}
					if (expiresAt < currentTime - threeDaysInMillis) {
						cookieManager.setCookie("https://*", name + "=; Expires=Thu, 01 Jan 1970 00:00:00 GMT");
					}
				}
			}
		}
		cookieManager.flush();
	}
	
	public void clearCookies() {
		cookieManager.removeAllCookies(null);
		cookieManager.flush();
	}
}


ชื่อไฟล์: com/studio/backend/domain/model/Site.java

package com.studio.backend.domain.model;

public class Site {
	private String name;
	private String url;
	
	public Site(String name, String url) {
		this.name = name;
		this.url = url;
	}
	
	public String getName() {
		return name;
	}
	
	public String getUrl() {
		return url;
	}
}

