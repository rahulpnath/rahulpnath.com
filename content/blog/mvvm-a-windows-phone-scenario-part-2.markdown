---
author: rahulpnath
comments: true
date: 2014-01-07 16:05:28+00:00
layout: post
slug: mvvm-a-windows-phone-scenario-part-2
title: MVVM – A Windows Phone Scenario – Part 2
wordpress_id: 771
categories:
- Windows Phone
tags:
- MVVM
- Windows Phone Series
---

We had looked into many of the common MVVM scenarios, that we come across while developing windows phone applications in [MVVM – A Windows Phone Scenario.](http://rahulpnath.com/blog/mvvm-a-windows-phone-scenario/) We will see some more that we were left off, in this post.

**5. Page Navigations and Parameters**

For almost all the application, we would need to transfer the control from one page to another, so that the user can navigate through the various contents on the application. In a MVVM application, these navigations would be basically triggered from the ViewModel, as it is there where we need to know where the next control should go to.
Navigation is basically a platform specific feature and we would not want to bring in any dependency between a platform specific feature and our ViewModels. So the best way here is to _inverse the dependency_, using an interface and inject the dependency via an IoC container. Will call the interface here as _INavigationService _as given below, and the implementation _NavigationService_.

``` csharp   
public interface INavigationService
{
	void Navigate(string uri);
	void Navigate(string uri, Dictionary<string, string> parameters);
	void PerformActionOnUIThread(Action action);
	void GoBack();
}

public class NavigationService : INavigationService
{
	public void Navigate(string uri)
	{
		DispatcherHelper.UIDispatcher.BeginInvoke(() =>
			{
				((PhoneApplicationFrame)Application.Current.RootVisual).Navigate(new Uri(uri, UriKind.Relative));
			});
	}

	public void Navigate(string uri, Dictionary<string, string> parameters)
	{
		StringBuilder uriBuilder = new StringBuilder();
		uriBuilder.Append(uri);
		if (parameters != null && parameters.Count > 0)
		{
			uriBuilder.Append("?");
			bool prependAmp = false;
			foreach (KeyValuePair<string, string> parameterPair in parameters)
			{
				if (prependAmp)
				{
					uriBuilder.Append("&");
				}

				uriBuilder.AppendFormat("{0}={1}", parameterPair.Key, parameterPair.Value);
				prependAmp = true;
			}
		}

		uri = uriBuilder.ToString();
		DispatcherHelper.UIDispatcher.BeginInvoke(() =>
		{
			((PhoneApplicationFrame)Application.Current.RootVisual).Navigate(new Uri(uri, UriKind.Relative));
		});
	}

	public void PerformActionOnUIThread(Action action)
	{
		DispatcherHelper.UIDispatcher.BeginInvoke(() => action.Invoke());
	}

	public void GoBack()
	{
		DispatcherHelper.UIDispatcher.BeginInvoke(() =>
		{
			((PhoneApplicationFrame)Application.Current.RootVisual).GoBack();
		});
	}
}
```

This is just a sample implementation that you can use. The _DispatcherHelper _used here is from MVVMLight, which needs to be initialized when the application starts. This can be in the App.xaml.cs in the application constructor

``` csharp
DispatcherHelper.Initialize();
```

Here in the above example the parameters are considered to be primitive data-types, which is why it is added as query parameters to the navigation uri. In case you want to have complex parameters passed between pages, you could have a property on the INavigationService, which can be set when calling the Naivgate method. This values can be retrieved when the OnNavigatedTo in the ViewModel.

``` csharp
public static object Parameters { get; set; }

public void Navigate(string uri, object parameters)
{
    Parameters = parameters;
    DispatcherHelper.UIDispatcher.BeginInvoke(() =>
        {
            ((PhoneApplicationFrame)Application.Current.RootVisual).Navigate(new Uri(uri, UriKind.Relative));
        });
}
```

The NavigationService needs to be registered with the MVVM IoC in ViewModelLocator (or anywhere else), and then you could either constructor inject it or create an instance in the BaseViewModel class, so that all ViewModels has a reference to this for navigation.

``` csharp
SimpleIoc.Default.Register<INavigationService, NavigationService>();
```

When developing applications for both Windows phone and Windows 8 or x-platform, your ViewModels would remain the same and the NavigationService implementations only would change, and would be accordingly injected into the IoC, when the application starts. So for any platform specific features/dependencies this would be the approach that you would need to choose to minimize the dependencies for your ViewModel. Other cases that I can think of right now is for Push Notifications, where each platform would have their own implementations for registering and raising notifications. So you would use the same approach to _inverse the dependencies._

**6. Page Events**

Most of the processing/data loading work in done usually on the OnNavigatedTo of the page. To hook to this event in the ViewModel, we would go ahead and introduce some base classes so that we can reuse this in all over ViewModels. We have a application specific base class for the PhoneApplicationPage. We override the _OnNavigatedTo_ event here and call on to the application specific base ViewModel’s OnNavigatedTo event. For any ViewModel to hook into this event just needs to override this method on the ViewModel, as shown in the sample below

``` csharp
public abstract class ApplicationPageBase: PhoneApplicationPage
{
    protected override void OnNavigatedTo(System.Windows.Navigation.NavigationEventArgs e)
    {
        base.OnNavigatedTo(e);

        (this.DataContext as ApplicationViewModelBase).OnPageNavigatedTo(this.NavigationContext.QueryString);

    }
}

public class ApplicationViewModelBase: ViewModelBase
{
    public virtual void OnPageNavigatedTo(Dictionary<string, string> parameters)
    {
        // Override this in any of the ViewModel to hook to the OnNavigatedTo event on the page
    }
}
```

Similarly for any of the other page events also you could create it in the page base class and call the corresponding function on the ViewModel base.

**7. Application Bar**

I have already put out a detailed post on how to implement an application bar using MVVM. It details out 2 approaches one using Messenger and another having the applciation bar as a service. Check it out at  - [Windows Phone Series – MVVM and ApplicationBar](http://rahulpnath.com/blog/windows-phone-series-mvvm-and-applicationbar/)

We have seen most of the basic scenarios that we normally come across while developing an application for windows phone and on how MVVM can be applied to that, so that we can make the best out of it. With lots of devices getting out there and having the need to have your application on all platform demands the maximum reuse, so that you can be out there quickly. MVVM plays a very important role in structuring your code making this possible. Even while developing cross platform application using [Xamarin](http://xamarin.com/), MVVM can be used to advantage so that all the application logic is neatly abstracted away from UI.

Do comment in on any of the missed scenarios that you normally come across while developing applications for the Windows platform.
