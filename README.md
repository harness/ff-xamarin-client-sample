# Example App using Harness Xamaring Binding SDK (ff-xamarin-client-sample) for Demo purposes

This Demo App will demonstrate the usage and capabilities of Xamarin client SDK for iOS and Android platform.

## Setup

Xamarin client uses Harness Binding library NuGet package for iOS and Android application.

To reference Android binding library add package:
```
Install-Package ff-android-xamarin-client-sdk -Version 0.5.2
```

To reference iOS binding library add package:
```
Install-Package ff-ios-xamarin-client-sdk -Version 0.5.0
```

## Usage

### iOS library

Library Initialization snippet
```

    //create configuration
    var config = new CfConfigurationProxy
    {
        StreamEnabled = true,
        AnalyticsEnabled = true,
        ConfigUrl = "https://config.feature-flags.uat.harness.io/api/1.0",
        EventUrl = "https://event.feature-flags.uat.harness.io/api/1.0",
        StreamUrl = "https://config.feature-flags.uat.harness.io/api/1.0/stream"
    };

    // set selected identifer
    var target = new CfTargetProxy
    {
        Identifier = "target_identifier",
        Name = "target_name"
    };

    // Initialize authentication. Update API_KEY with your key.
    CfClientProxy.Shared.InitializeWithApiKey(API_KEY, config, target);
```

Subscribing on receiving library events.

```
using System;
using ff_ios_client_sdk_proxy;

public class CfListener : CfClientDelegate
{

    public CfListener()
    {
        // Subscribe on getting events from native iOS library
        CfClientProxy.Shared.Delegate = this;
    }
    // Received in case of error
    public override void OnErrorWithError(CfErrorProxy error){}

    // Contains array with flags received each pooling interval
    public override void OnPollingEventReceivedWithEvaluations(CxEvaluation[] evaluations){}

    // Event when flag value is changed
    public override void OnStreamEventReceivedWithEvaluation(CxEvaluation evaluation){}

    // Message received from library event
    public override void OnMessageReceivedWithMessage(CxMessage message){}

    // On Stream opened event
    public override void OnStreamOpened() {}

    // On Stream closed event
    public override void OnStreamCompleted() {}
}
```

### Android library

Library Initialization snippet

```
    CfConfiguration configuration = new CfConfiguration.Builder()
        .EnableStream(true)
        .EnableAnalytics(true)
        .EventUrl("https://event.feature-flags.uat.harness.io/api/1.0")
        .StreamUrl("https://config.feature-flags.uat.harness.io/api/1.0/stream")
        .BaseUrl("https://config.feature-flags.uat.harness.io/api/1.0")
        .Build();

    Target target = new Target()
        .InvokeName(account)
        .InvokeIdentifier(account);

     // Initialize authentication. Update API_KEY with your key. Listener object should implement IAuthCallback interface.
    CfClient.Instance.Initialize(this.context, API_KEY, configuration, target, listener);
```

Subscribing on receiving library events.

```
using System;
using System.Linq;
using IO.Harness.Cfsdk.Cloud.Events;
using IO.Harness.Cfsdk.Cloud.Model;
using IO.Harness.Cfsdk.Cloud.Oksse;
using IO.Harness.Cfsdk.Cloud.Oksse.Model;
using Java.Interop;
using Java.Util;
using IO.Harness.Cfsdk;

public class CfListener : Java.Lang.Object, IAuthCallback, IEventsListener
{

    public CfListener()
    {
         // Subscribe on getting events from native Android library
        CfClient.Instance.RegisterEventsListener(this);
    }
    public void AuthorizationSuccess(AuthInfo p0, AuthResult p1)
    {
        // p1.Success contains status of authorization
        // In case of error p1.Error contains error message
    }
    public void OnEventReceived(StatusEvent p0)
    {
        var eventType = p0.EventType;
        if(StatusEvent.EVENT_TYPE.SseStart == eventType)
        {
            // Stream started
        }
        else if( StatusEvent.EVENT_TYPE.SseEnd == eventType)
        {
            // Stream Ended
        }
        else if (StatusEvent.EVENT_TYPE.EvaluationChange == eventType)
        {
            Java.Lang.Object payload = p0.ExtractPayload();
            var ev = payload as IO.Harness.Cfsdk.Cloud.Core.Model.Evaluation;
            // Flag changed event
        }
        else if (StatusEvent.EVENT_TYPE.EvaluationReload == eventType)
        {
            Java.Lang.Object payload = p0.ExtractPayload();

            var t = payload.JavaCast<ArrayList>();
            var arr = t.ToEnumerable<IO.Harness.Cfsdk.Cloud.Core.Model.Evaluation>().ToArray();

            // Each pulling interval we will receive array of available flags.

        }
    }

}
```
