# Raheel
Project Portfolios


import android.app.Activity;
import android.app.AlertDialog;
import android.content.BroadcastReceiver;
import android.content.ComponentName;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.IntentFilter;
import android.content.SharedPreferences;
import android.net.ConnectivityManager;
import android.net.Uri;
import android.os.Bundle;
import android.provider.Settings;
import android.support.annotation.RequiresApi;
import android.support.p000v4.app.ActivityCompat;
import android.support.p000v4.content.ContextCompat;
import android.support.p003v7.app.AppCompatActivity;
import android.support.p003v7.widget.PopupMenu;
import android.support.p003v7.widget.Toolbar;
import android.view.MenuItem;
import android.view.View;
import android.view.inputmethod.InputMethodManager;
import android.widget.ImageView;
import android.widget.RelativeLayout;
import android.widget.TextView;
import com.Keyboard.ArabicKeyboard.helper.UncachedInputMethodManagerUtils;
import com.Keyboard.ArabicKeyboard.ime.SpanishIME;
import com.Keyboard.ArabicKeyboard.service.MyService;
import com.google.android.gms.ads.AdListener;
import com.google.android.gms.ads.AdRequest;
import com.google.android.gms.ads.AdView;
import com.voicetyping.arabickeyboard.speech.dictation.recognition.R;
import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {
    public static final String PREFS_NAME = "amharickeyboard";
    public static final int REQUEST_ID_MULTIPLE_PERMISSIONS = 68;
    private static Activity activity = null;
    public static boolean isenabled_voice = false;
    public ImageView about;
    public AlertDialog alertDialog;
    Analytics analytics;
    boolean exit_msg;
    public RelativeLayout finishButton;
    /* access modifiers changed from: private */
    public AdView mAdView;
    private InputMethodManager mImm;
    public ImageView mMenu;
    private InputMethodChangeReceiver mReceiver;
    public ImageView rate;
    public RelativeLayout settingsButton;
    int setup_step = 1;
    public SharedPreferences sharedPreferences;

    /* access modifiers changed from: protected */
    public void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView((int) R.layout.activity_main);
        checkAndRequestPermissions();
        this.analytics = new Analytics(this);
        this.analytics.sendScreenAnalytics("MainActivity");
        this.mReceiver = new InputMethodChangeReceiver();
        registerReceiver(this.mReceiver, new IntentFilter("android.intent.action.INPUT_METHOD_CHANGED"));
        setSupportActionBar((Toolbar) findViewById(R.id.toolbar));
        this.sharedPreferences = getSharedPreferences(PREFS_NAME, 0);
        activity = this;
        this.mImm = (InputMethodManager) getSystemService("input_method");
        this.settingsButton = (RelativeLayout) findViewById(R.id.setting_button);
        this.finishButton = (RelativeLayout) findViewById(R.id.finish_button);
        this.settingsButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                if (MainActivity.this.setup_step == 1) {
                    MainActivity.this.enableKeyBoard();
                    Intent intent = new Intent(MainActivity.this, MyService.class);
                    MainActivity.this.analytics.sendEventAnalytics("Enable", "Tapped");
                    MainActivity.this.startService(intent);
                    return;
                }
                ((InputMethodManager) MainActivity.this.getSystemService("input_method")).showInputMethodPicker();
            }
        });
        this.finishButton.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                MainActivity.this.finish();
                MainActivity.this.analytics.sendEventAnalytics("Done", "Tapped");
            }
        });
        this.mMenu = (ImageView) findViewById(R.id.toolBar_image);
        this.about = (ImageView) findViewById(R.id.toolBar_about);
        this.rate = (ImageView) findViewById(R.id.toolBar_rate);
        this.about.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                MainActivity.this.startActivity(new Intent(MainActivity.this, AboutUSActivity.class));
                MainActivity.this.analytics.sendEventAnalytics("Aboutus", "Tapped");
            }
        });
        this.rate.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                MainActivity.this.rateApp();
                MainActivity.this.analytics.sendEventAnalytics("Rate", "Tapped");
            }
        });
        this.mMenu.setOnClickListener(new View.OnClickListener() {
            public void onClick(View view) {
                MainActivity.this.rateApp();
            }
        });
    }

    private void showMenu() {
        PopupMenu popupMenu = new PopupMenu(this, this.mMenu);
        popupMenu.getMenuInflater().inflate(R.menu.menu_main, popupMenu.getMenu());
        popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
            public boolean onMenuItemClick(MenuItem menuItem) {
                int itemId = menuItem.getItemId();
                if (itemId == R.id.about_us) {
                    MainActivity.this.startActivity(new Intent(MainActivity.this, AboutUSActivity.class));
                    return true;
                } else if (itemId == R.id.privacy_policy) {
                    MainActivity.this.privacyPolicy();
                    return true;
                } else if (itemId == R.id.rate_us) {
                    MainActivity.this.rateApp();
                    return true;
                } else if (itemId != R.id.share) {
                    return true;
                } else {
                    MainActivity.this.shareApp(MainActivity.this.getString(R.string.share_message));
                    return true;
                }
            }
        });
        popupMenu.show();
    }

    public boolean isInputMethodEnabled() {
        return new ComponentName(this, SpanishIME.class).equals(ComponentName.unflattenFromString(Settings.Secure.getString(getContentResolver(), "default_input_method")));
    }

    public void rateApp() {
        startActivity(new Intent("android.intent.action.VIEW", Uri.parse("market://details?id=" + getPackageName())));
    }

    public void shareApp(String str) {
        Intent intent = new Intent("android.intent.action.SEND");
        intent.setType("text/plain");
        intent.putExtra("android.intent.extra.SUBJECT", "Voice Keyboard ");
        intent.putExtra("android.intent.extra.TEXT", str);
        startActivity(Intent.createChooser(intent, "Share via"));
    }

    public void privacyPolicy() {
        startActivity(new Intent("android.intent.action.VIEW", Uri.parse("")));
    }

    public class InputMethodChangeReceiver extends BroadcastReceiver {
        public InputMethodChangeReceiver() {
        }

        @RequiresApi(api = 16)
        public void onReceive(Context context, Intent intent) {
            if (intent.getAction().equals("android.intent.action.INPUT_METHOD_CHANGED")) {
                if (MainActivity.this.isInputMethodEnabled()) {
                    MainActivity.this.setup_step = 3;
                } else {
                    MainActivity.this.setup_step = 2;
                }
                MainActivity.this.setScreen();
            }
        }
    }

    /* access modifiers changed from: protected */
    public void onDestroy() {
        unregisterReceiver(this.mReceiver);
        super.onDestroy();
    }

    public void enableKeyBoard() {
        startActivityForResult(new Intent("android.settings.INPUT_METHOD_SETTINGS"), 0);
    }

    private void checkKeybordExit() {
        if (UncachedInputMethodManagerUtils.isThisImeEnabled(this, this.mImm)) {
            this.setup_step = 2;
            if (isInputMethodEnabled()) {
                this.setup_step = 3;
                return;
            }
            return;
        }
        this.setup_step = 1;
    }

    /* access modifiers changed from: protected */
    @RequiresApi(api = 16)
    public void onResume() {
        super.onResume();
        initializeAds();
        ((ImageView) findViewById(R.id.toolBar_image)).setVisibility(0);
        checkKeybordExit();
        setScreen();
        ((TextView) findViewById(R.id.toolBar_title)).setText("Arabic Voice Typing");
    }

    private void initializeAds() {
        if (isNetworkConnected()) {
            new AdRequest.Builder().build();
            this.mAdView = (AdView) findViewById(R.id.adView);
            this.mAdView.setVisibility(0);
            this.mAdView.loadAd(new AdRequest.Builder().build());
        } else {
            this.mAdView = (AdView) findViewById(R.id.adView);
            this.mAdView.setVisibility(8);
        }
        this.mAdView.setAdListener(new AdListener() {
            public void onAdClosed() {
                super.onAdClosed();
            }

            public void onAdFailedToLoad(int i) {
                super.onAdFailedToLoad(i);
                MainActivity.this.mAdView.setVisibility(8);
            }

            public void onAdLeftApplication() {
                super.onAdLeftApplication();
            }

            public void onAdOpened() {
                super.onAdOpened();
            }

            public void onAdLoaded() {
                super.onAdLoaded();
            }
        });
    }

    /* access modifiers changed from: private */
    @RequiresApi(api = 16)
    public void setScreen() {
        switch (this.setup_step) {
            case 1:
                setupScreen_1();
                return;
            case 2:
                setupScreen_2();
                return;
            case 3:
                setupScreen_3();
                return;
            default:
                return;
        }
    }

    public static void finishActivity() {
        activity.finish();
    }

    private boolean isNetworkConnected() {
        return ((ConnectivityManager) getSystemService("connectivity")).getActiveNetworkInfo() != null;
    }

    public void onBackPressed() {
        exitAlert();
    }

    public void exitAlert() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Rate Us");
        builder.setMessage("We would like to know your experience, please give us your feedback.");
        builder.setPositiveButton("YES", new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialogInterface, int i) {
                MainActivity.this.rateApp();
            }
        });
        builder.setNegativeButton("NO", new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialogInterface, int i) {
                MainActivity.this.finish();
            }
        });
        this.alertDialog = builder.create();
        this.alertDialog.show();
    }

    @RequiresApi(api = 16)
    private void setupScreen_1() {
        findViewById(R.id.step_1).setBackground(getResources().getDrawable(R.drawable.selected_page_drawable));
        findViewById(R.id.step_2).setBackground(getResources().getDrawable(R.drawable.unseleted_page_drawable));
        findViewById(R.id.step_3).setBackground(getResources().getDrawable(R.drawable.unseleted_page_drawable));
        ((TextView) findViewById(R.id.instructions_text_top)).setText(R.string.step_1_instructions);
        ((TextView) findViewById(R.id.setting_button_text)).setText("Settings");
        this.analytics.sendEventAnalytics("Settings", "Tapped");
        ((TextView) findViewById(R.id.instructions_text_bottom)).setVisibility(8);
        this.finishButton.setVisibility(8);
    }

    @RequiresApi(api = 16)
    private void setupScreen_2() {
        findViewById(R.id.step_1).setBackground(getResources().getDrawable(R.drawable.unseleted_page_drawable));
        findViewById(R.id.step_2).setBackground(getResources().getDrawable(R.drawable.selected_page_drawable));
        findViewById(R.id.step_3).setBackground(getResources().getDrawable(R.drawable.unseleted_page_drawable));
        ((TextView) findViewById(R.id.instructions_text_top)).setText(R.string.step_2_instructions);
        ((TextView) findViewById(R.id.setting_button_text)).setText("Activate");
        this.analytics.sendEventAnalytics("Activate", "Tapped");
        ((TextView) findViewById(R.id.instructions_text_bottom)).setVisibility(8);
        this.finishButton.setVisibility(8);
    }

    @RequiresApi(api = 16)
    private void setupScreen_3() {
        findViewById(R.id.step_1).setBackground(getResources().getDrawable(R.drawable.unseleted_page_drawable));
        findViewById(R.id.step_2).setBackground(getResources().getDrawable(R.drawable.unseleted_page_drawable));
        findViewById(R.id.step_3).setBackground(getResources().getDrawable(R.drawable.selected_page_drawable));
        ((TextView) findViewById(R.id.instructions_text_top)).setText(R.string.disable_message);
        ((TextView) findViewById(R.id.setting_button_text)).setText("Disable ");
        this.settingsButton.setBackground(getResources().getDrawable(R.drawable.disable));
        ((TextView) findViewById(R.id.instructions_text_bottom)).setVisibility(0);
        ((TextView) findViewById(R.id.instructions_text_bottom)).setText(R.string.step_3_instructions);
        this.finishButton.setVisibility(0);
    }

    private boolean checkAndRequestPermissions() {
        int checkSelfPermission = ContextCompat.checkSelfPermission(this, "android.permission.RECORD_AUDIO");
        int checkSelfPermission2 = ContextCompat.checkSelfPermission(this, "android.permission.WRITE_EXTERNAL_STORAGE");
        int checkSelfPermission3 = ContextCompat.checkSelfPermission(this, "android.permission.READ_EXTERNAL_STORAGE");
        ArrayList arrayList = new ArrayList();
        if (checkSelfPermission != 0) {
            arrayList.add("android.permission.RECORD_AUDIO");
        }
        if (checkSelfPermission2 != 0) {
            arrayList.add("android.permission.WRITE_EXTERNAL_STORAGE");
        }
        if (checkSelfPermission3 != 0) {
            arrayList.add("android.permission.READ_EXTERNAL_STORAGE");
        }
        if (arrayList.isEmpty()) {
            return true;
        }
        ActivityCompat.requestPermissions(this, (String[]) arrayList.toArray(new String[arrayList.size()]), 68);
        return false;
    }
}


 Twitter  Facebook  Stumbleupon  LinkedIn
Select a decompiler
 Jadx decompiler for Android
Privacy Policy
