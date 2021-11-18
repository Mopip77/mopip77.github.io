---
layout: post
title: PT plugin plus chrome安装
tags:
- mac
---

pt plugin 由于被Chrome移除了，无法通过chrome商店下载，因此记录一下安装方式

通过常规的zip或crx安装会在chrome重启之后，被chrome自动移除

下面的两种安装方式通过设置`Chrome Extension白名单`实现永久安装

## Windows

windows的安装比较简单，官方也给出了安装手册，[windows 通过crx安装](https://github.com/ronggang/PT-Plugin-Plus/wiki/install-from-crx)

## MAC

mac需要写入一个配置文件，起名为`com.google.Chrome.mobileconfig`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>PayloadContent</key>
    <array>
        <dict>
            <key>PayloadContent</key>
            <dict>
                <key>com.google.Chrome</key>
                <dict>
                    <key>Forced</key>
                    <array>
                        <dict>
                            <key>mcx_preference_settings</key>
                            <dict>
                                <key>ExtensionInstallWhitelist</key>
                                <array>
                                    <string>dmmjlmbkigbgpnjfiimhlnbnmppjhpea</string>
                                </array>
                            </dict>
                        </dict>
                    </array>
                </dict>
            </dict>
            <key>PayloadEnabled</key>
            <true/>
            <key>PayloadIdentifier</key>
            <string>MCXToProfile.7e2bec75-299e-44ff-b405-628007abffff.alacarte.customsettings.bdac4880-d25f-4cdd-8472-05473f005e7e</string>
            <key>PayloadType</key>
            <string>com.apple.ManagedClient.preferences</string>
            <key>PayloadUUID</key>
            <string>bdac4880-d25f-4cdd-8472-05473f005e7e</string>
            <key>PayloadVersion</key>
            <integer>1</integer>
        </dict>
    </array>
    <key>PayloadDescription</key>
    <string>Included custom settings:
com.google.Chrome
</string>
    <key>PayloadDisplayName</key>
    <string>MCXToProfile: com.google.Chrome</string>
    <key>PayloadIdentifier</key>
    <string>com.google.Chrome</string>
    <key>PayloadOrganization</key>
    <string></string>
    <key>PayloadRemovalDisallowed</key>
    <true/>
    <key>PayloadScope</key>
    <string>System</string>
    <key>PayloadType</key>
    <string>Configuration</string>
    <key>PayloadUUID</key>
    <string>7e2bec75-299e-44ff-b405-628007abffff</string>
    <key>PayloadVersion</key>
    <integer>1</integer>
</dict>
</plist>
```

写入之后，双击该文件安装即可

然后通过crx安装插件