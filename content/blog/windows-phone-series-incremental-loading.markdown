---
author: rahulpnath
comments: true
date: 2013-03-03 08:53:45+00:00
layout: post
slug: windows-phone-series-incremental-loading
title: Windows Phone Series – Incremental Loading
wordpress_id: 471
categories:
- .Net
- Windows Phone
- WP7
tags:
- 500px
- 500px infinite scrolling
- 500px windows phone
- infinite loading
- infinite scrolling windows phone
- LongListSelector scrolling
- metro app
- metro paged loading
- Windows Phone 8
- Windows Phone Series
---

Some time back we had a look on doing [Incremental Loading with a Windows 8 store app](http://rahulpnath.wordpress.com/2012/10/28/windows-8-series-incremental-loading/). This same scenario is something that one would come across quite frequently while developing a Windows Phone application too. We have a couple of options in dealing with this while on a Windows phone application. In an ideal case while binding to a large data on a windows phone application, we might be using a [LongListSelector](http://msdn.microsoft.com/en-US/library/windowsphone/develop/microsoft.phone.controls.longlistselector(v=vs.105).aspx) or a Listbox control.

While using a LongListSelector, we can use the Link event(if you are using [Windows Phone 7.1 toolkit](http://phone.codeplex.com/)) Or the [ItemRealized](http://msdn.microsoft.com/en-US/library/windowsphone/develop/microsoft.phone.controls.longlistselector.itemrealized(v=vs.105).aspx) event(if you are on Windows phone 8.0). Basically we would be doing the same thing in either of these cases, checking the current item that is getting realized and see what is the index of the item in the whole list of data that you have currently and check if its time for you to fetch the next set of data from your data source(possibly a web service). As usual for the sample we will be using the [500px](http://500px.com/popular) api.

Below is the piece of code that will fetch us the photo from the [500px api](http://developers.500px.com/) to populate the listbox data.

``` csharp
private static int requestPerPage = 20;
private int currentPage = 1;
private bool isCurrentlyLoading = false;

private ObservableCollection<Photo> Photos = new ObservableCollection<Photo>();

private string datasourceUrl="https://api.500px.com/v1/photos?feature=popular&consumer_key=" 
          + consumerKey + "&rpp=" + requestPerPage.ToString() + "&page={0}";

private void LoadDataFromSource()
{
    isCurrentlyLoading = true;
    var query = string.Format(datasourceUrl, currentPage);
    WebClient client = new WebClient();
    client.DownloadStringCompleted += client_DownloadStringCompleted;
    client.DownloadStringAsync(new Uri(query));

}

void client_DownloadStringCompleted(object sender, DownloadStringCompletedEventArgs e)
{
    using (var reader = new MemoryStream(Encoding.Unicode.GetBytes(e.Result)))
    {
        var ser = new DataContractJsonSerializer(typeof(RootObject));
        RootObject obj = (RootObject)ser.ReadObject(reader);
        currentPage = obj.current_page + 1;
        if (obj != null)
        {
            this.Dispatcher.BeginInvoke(() =>
                {
                    foreach (var photo in obj.photos)
                    {
                        Photos.Add(photo);
                    }
                    isCurrentlyLoading = false;
                });
        }
    }
}
```

In the ItemRealized/Link event based on whether you are developing for Windows Phone 7 or 8, below would be the code that goes into that. We need to check the current item that is realized i.e rendered on the UI and see if it is

``` csharp
private void photosList_ItemRealized_1(object sender, ItemRealizationEventArgs e)
{
    Photo photo = e.Container.Content as Photo;
    if (photo != null)
    {
        int offset = 2;
        // Only if there is no data that is currently getting loaded
        // would be initiate the loading again
        if (!isCurrentlyLoading && Photos.Count - Photos.IndexOf(photo) <= offset)
        {
            LoadDataFromSource();
        }
    }
}
```

In case you want to use a normal listbox, you can do that also. We would need to hook up with VisualStateGroups. This [link](http://blogs.msdn.com/b/slmperf/archive/2011/06/30/windows-phone-mango-change-listbox-how-to-detect-compression-end-of-scroll-states.aspx) explains this in details, and I have just reused parts of it as is. We need to override the scrollviewer style to hook into this new StateGroups. We need to look for CompressionBottom state, for the currentstatechanged event of the scrollviewer.

``` csharp
private void myScrollViewer_Loaded_1(object sender, RoutedEventArgs e)
{
    SetScrollViewer();
}

private void SetScrollViewer()
{
    // Visual States are always on the first child of the control template 
    FrameworkElement element = VisualTreeHelper.GetChild(myScrollViewer, 0) 
                            as FrameworkElement;
    if (element != null)
    {
        VisualStateGroup vgroup = FindVisualState(element, "VerticalCompression");

        if (vgroup != null)
        {
            vgroup.CurrentStateChanging += 
                 new EventHandler<VisualStateChangedEventArgs>(vgroup_CurrentStateChanging);
        }
    }
}

private void vgroup_CurrentStateChanging(object sender, VisualStateChangedEventArgs e)
{
    if (e.NewState.Name == "CompressionTop")
    {

    }

    if (e.NewState.Name == "CompressionBottom")
    {
        if (!isCurrentlyLoading )
        {
            LoadDataFromSource();
        }
    }
    if (e.NewState.Name == "NoVerticalCompression")
    {

    }
}
```

You can use either of these two ways to incrementally load data on a windows phone app. The entire code that is used in this blog is avaialble [here](https://github.com/rahulpnath/IncrementalLoadingPhone). The sample app shows both these methods. The pivot header specifies the method that is used, LongListSelector and Scrollstates



![windows phone incremental loading using longlistselector](/images/wp_incremental_loading_longlistselector.jpg)![windows phone incremental loading using scrollstates](/images/wp_incremental_loading_scrollstates.jpg)

Hope this helps you to incrementally load data on your windows phone application

[Download Code](https://github.com/rahulpnath/IncrementalLoadingPhone)
