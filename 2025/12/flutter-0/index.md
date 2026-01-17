# Flutter 0


<!--more-->

## flutter的网络配置
### mac 
`macos/Runner/Info.plist`
``` xml
	<key>NSAppTransportSecurity</key>
		<dict>
			<key>NSAllowsLocalNetworking</key>
			<true /> <!-- 允许本地网络 -->
			<key>NSAllowsArbitraryLoads</key>
			<true /> <!-- 允许任意 HTTP（仅开发环境） -->
	</dict>
```

`macos/Runner/DebugProfile.entitlements`

``` xml
		<key>com.apple.security.network.client</key>
		<true />
```

`macos/Runner/Release.entitlements`

``` xml
	<key>com.apple.security.network.client</key>
	<true />
```

