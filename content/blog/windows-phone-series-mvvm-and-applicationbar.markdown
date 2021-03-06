---
author: rahulpnath
comments: true
date: 2013-04-17 11:04:01+00:00
layout: post
slug: windows-phone-series-mvvm-and-applicationbar
title: Windows Phone Series – MVVM and ApplicationBar
wordpress_id: 520
categories:
- .Net
- Windows Phone
- WP7
tags:
- Appbar and mvvm
- MVVM
- windows phone 7
- Windows Phone 8
- Windows phone applicationBar
- Windows Phone BindableApplicationBar
- Windows Phone Series
---

ApplicationBar on a windows phone, is to provide users of your app with quick access to the most commonly used tasks. For a mail app this would be refresh/new mail, for a photo app it might be like/unlike button, settings etc are common buttons that appear in an ApplicationBar . But then not always do we have them as static, and we would want to add/remove icons/menu items to that based on the applications current state.For example, you would want to add the like/unlike button only if the user is logged in, or even the settings icon would be only for logged in users. Whatever might be your scenario, if you are looking for to add/remove icons from the application bar from your code then this article is for you.

 

In a normal app which uses code-behind, this can be easily done by accessing the ‘ApplicationBar’ in the code-behind class like below

``` csharp
ApplicationBar.Buttons.Add(<your button>);
```


When you are using MVVM you would want to do this from your ViewModel(VM). Below are the approaches that you could use to achieve the same





**1. Mesenger Service**





Whenever we need to communicate between VM’s or between your view model and View then we would want to do that in the most decoupled manner. When using [MVVMLight](http://www.galasoft.ch/mvvm/), we could use the Messenger class to achieve this. We would send a message indicating that we want to add a new appbar button from the ViewModel, and the View code behind, which already has registered for such an event would get notified and add the icon for us .





Below is how the Code-behind would look like . We register for a NotificationMessage(you could also use your own notification class for this), and see what kind of button needs to be added and adds that to the ApplicationBar. On click of the appbar button, we wire up the click event to a command of the ViewModel. Though there is some code behind in here, we are not going away from MVVM here, as we still have clear separation of concerns and also testability is not affected.

``` csharp
    public partial class MainPage : PhoneApplicationPage
    {
        public MainViewModel viewModel
        {
            get
            {
                return this.DataContext as MainViewModel;
            }
        }
    
        private ApplicationBarIconButton settingsButton;
        // Constructor
        public MainPage()
        {
            InitializeComponent();
            settingsButton = new ApplicationBarIconButton()
            {
                Text = "Settings",
                IconUri = new Uri("Images/appbar.feature.settings.rest.png", UriKind.Relative)
            };
            settingsButton.Click += settingsButton_Click;
            // Register for the messenger 
            Messenger.Default.Register<NotificationMessage>(this, OnNotificationMessage);
        }
    
        void settingsButton_Click(object sender, EventArgs e)
        {
            viewModel.SettingsCommand.Execute(null);
        }
    
        private void OnNotificationMessage(NotificationMessage message)
        {
            // Check here for the notification
            // You can also build cutoms notification message here for this by inheriting from MessageBase
            if (message.Notification == "AddSettings")
            {
                if (ApplicationBar == null)
                {
                    ApplicationBar = new ApplicationBar();
                }
                ApplicationBar.Buttons.Add(settingsButton);
            }
        }
    }

```



****





**2. ApplicationBar Service**





Like we use [NavigationService](http://www.geekchamp.com/articles/mvvm-in-real-life-windows-phone-applications-part2), for navigating from VM’s we could also create a ApplicationBarService, that can be used to add application bar icons from ViewModels. For this I have created a base class, MyModelBase, for all my VM’s which inturn inherits from ViewModelBase of MVVMLight. This base class holds an interface for the ApplicationBarService.

``` csharp
    public class MyModelBase: ViewModelBase
    {
        public IApplicationBarService ApplicationBar { get; set; }
    
        public MyModelBase()
        {
    
        }
        public MyModelBase(IApplicationBarService appBar)
        {
            ApplicationBar = appBar;
        }
    }

```



The interface IApplicationBarService, would have the functions that we would want to Add/Remove icons from the application bar. For now I have just put in the AddButton. You could also add RemoveButton and any other things that you would want in there.

``` csharp
    public interface IApplicationBarService
    {
        IApplicationBar ApplicationBar { get;} 
    
        void AddButton(string title, Uri imageUrl, Action OnClick);
    }
```

Implementation for this interface is as below

``` csharp
    public class ApplicationBarService: IApplicationBarService
    {
        public void AddButton(string title, Uri imageUrl, Action OnClick)
        {
            ApplicationBarIconButton newButton = new ApplicationBarIconButton()
                {
                    Text = title, 
                    IconUri = imageUrl, 
                };
            newButton.Click += ((sender,e) => {OnClick.Invoke();}) ;
    
            ApplicationBar.Buttons.Add(newButton);
           
        }
    
        public IApplicationBar ApplicationBar
        {
            get
            {
                var currentPage = ((App)Application.Current).RootFrame.Content as PhoneApplicationPage;
                if (currentPage.ApplicationBar == null)
                {
                    currentPage.ApplicationBar = new ApplicationBar();
                }
                return currentPage.ApplicationBar;
            }
        }
    }
```

The ApplicationBar property reads gets the current ApplicationBar from the current page. If it is not defined then it would simply create a new one. The Add function just adds a new button and wires up the click event of the button, to the function that is passed in by the VM. We could also use Commands here, for now I just wanted to keep it simple





In cases where you don’t want to add buttons dynamically, but just have static buttons you could also use BindableApplicationBar implementations that are there. One such implementation is there along with the [Phone7.Fx](http://phone7.codeplex.com/) library. There are also many other implementations for the same.





Hope this helps you to decouple your application bar icons from the ViewModel.You can find a [sample](https://github.com/rahulpnath/Blog/tree/master/PhoneAppBarMvvm) implementation for this. In the sample both these approaches are shown for adding icons. You could figure out the Remove pretty easily.





![windows phone applicationbar mvvm](/images/wp_applicationbar_icon_mvvm.png)





Hope it helps!!
