# sigma-ssai-exo-sdk
### Import SDK

Tải xuống [sigma-dai-exo-ssai-sdk](https://github.com/truongnguyen1804/sigma-ssai-exo-sdk/blob/main/sigma-dai-exo-ssai/sigma-dai-exo-ssai.aar) về dự án của bạn và cấu hình nó như một thư viện

#### Example

##### Bước 1:

Có hai cách cài đặt sdk:

###### Cách 1: Khởi tạo phiên hoạt động trong SDK (Init Session)

Sau khi khởi tạo phiên hoạt động trong SDK, SDK sẽ trả về một SourceUrl --> configPlayer(url)

```
AdsTracking.getInstance().initSession(urlSSAI, new AdsTracking.InitSessionListener() {
            @Override
            public void onResponse(int code, String url) {
                PlayActivity.this.runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        configPlayer(url);
                    }
                });
            }
            @Override
            public void onError(int code) {
                Log.e(TAG, "Code " + code);
            }

        });
```

###### Cách 2: SourceUrl và TrackingUrl có sẵn

```
AdsTracking.getInstance().setUrlTracking(urlTracking);
configPlayer(urlSource);
```



##### Bước 2:

###### Cung cấp thời gian cho SDK từ player qua AdsTracking.TrackingParams

Khởi tạo một giao diện AdsTracking.TrackingParams trackingParams:

```
trackingParams = new AdsTracking.TrackingParams() {
    @Override
    public long onTimeUpdate() {
        if (exoPlayer != null) {
            if (exoPlayer.isPlayingAd())
                return -1;
            if (exoPlayer.isCurrentMediaItemDynamic()) {
                // live
                if (windowLive == null) {
                    windowLive = new Timeline.Window();
                }
                if (periodLive == null) {
                    periodLive = new Timeline.Period();
                }
                if (!exoPlayer.getCurrentTimeline().isEmpty()) {
                    long positionInPeriod = exoPlayer.getCurrentPosition(); // in period
                    long position = positionInPeriod;
                    Timeline.Period p = exoPlayer.getCurrentTimeline().getPeriod(exoPlayer.getCurrentPeriodIndex(), periodLive);
                    long positionInWindows = p.getPositionInWindowMs();
                    position -= positionInWindows;
                    return position;
                }
            } else {
                // vod
                long position = exoPlayer.getCurrentPosition();
                return position;
            }
        }
        return -1;
    }
};
```

Trong đó `onTimeUpdate()` cần trả về thời gian của player

##### Bước 3: 

Lắng nghe sự kiện sẵn sàng phát của player

```
@Override
public void onPlaybackStateChanged(int playbackState) {
    if (playbackState == ExoPlayer.STATE_READY) {
        AdsTracking.getInstance().startTracking();
    }

}
```

Bắt đầu tracking bằng cách gọi `AdsTracking.getInstance().startTracking()` khi sự kiện phát lại của player sẵn sàng.



##### Chú ý:

Gọi `stopTracking()` khi player gặp sự cố

```
@Override
public void onPlayerError(PlaybackException error) {
    AdsTracking.getInstance().stopTracking();
}
```

Gọi `destroy()`  khi PlayActivity bị phá huỷ

```
@Override
protected void onDestroy() {
    AdsTracking.getInstance().destroy();
    super.onDestroy();
}
```
