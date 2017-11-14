Logging
=======

Log Location
    /Library/Logs/apsd.log

Commands::

    sudo touch /Library/Logs/apsd.log
    sudo defaults write /Library/Preferences/com.apple.apsd APSWriteLogs -bool TRUE
    sudo defaults write /Library/Preferences/com.apple.apsd APSLogLevel -int 7
    sudo killall apsd


Per Topic status can be retrieved by executing::

    /System/Library/PrivateFrameworks/ApplePushService.framework/apsctl status

