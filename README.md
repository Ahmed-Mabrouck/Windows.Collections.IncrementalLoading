# Windows.Collection.IncrementalLoading
A Windows Runtime custom collection that supports observability and incremental loading out of the box.

# What can IncrementalLoadingCollection do?

It can be considered a custom type of generic collections that can apply paging to huge amount of data in another source (Database Entities, Web Service Output, Other Collections, etc...)

It Supports out of the box:

Observability: all changes inside in the collection are observed by binding clients.

Incremental Loading: supports collection items paging by loading more data on scrolling data items control event trigger.


# How to get it?

There are 2 ways to use Windows.Collections.IncrementalLoading:

01.Follow NuGet installation process: https://www.nuget.org/packages/Windows.Collections.IncrementalLoading/.

02.Search for Windows Incremental Loading Collection on Visual Studio Manage NuGet Packages and click Install button.


# How to use it?

If I have a data source which is a List<int> contains 50 numbers (from 1 to 50) like following:

```C#
    private List<int> source = new List<int>
        {
            1,2,3,4,5,6,7,8,9,10,
            11,12,13,14,15,16,17,18,19,20,
            21,22,23,24,25,26,27,28,29,30,
            31,32,33,34,35,36,37,38,39,40,
            41,42,43,44,45,46,47,48,49,50
        };
```

I want to bind this List<int> to a ListView and I want to apply paging to these data, each page contains 5 items. 

On scrolling the ListView, one more page will be loaded to the ListView.

Example: 

The ListView will contain initially the first 5 number (1, 2, 3, 4, 5).

On scrolling, another page (5 numbers) will be fetched and then the ListView will containt the first 10 number (1, 2, 3, 4, 5, 6, 7, 8, 9, 10) and so on.

This behavior can be done by attaching an IncrementalLoadingCollection to the Source collection using the coming steps.

01.I need to declare a method that returns the items of a certain page, this is a standard that I alway use for that using Linq:

```C#
    private async System.Threading.Tasks.Task<IEnumerable<int>> GetPageItems(int currentPage, int pageSize)
    {
        return Source.Skip(currentPage * pageSize).Take(pageSize);
    }
```

To supress the warning and run the method asynchronously (which is better for UI responsivness), you can wrap your code inside a Task block, it will look lik that:

```C#
    private async System.Threading.Tasks.Task<IEnumerable<int>> GetPageItems(int currentPage, int pageSize)
    {
        return await System.Threading.Tasks.Task.Run<IEnumerable<int>>(() =>
            {
                return Source.Skip(currentPage * pageSize).Take(pageSize);
            });
    }
```

*Note: inside GetPageItems methods you can call any awaitable taks (httpclient requests, openning files, connecting to database, etc..).

02.Declare IncrementalLoadingCollection and initialize it with GetPageItems method and 5 which is the page size (Number of items per page):

```C#
    public class Class
    {
        public Windows.Collections.IncrementalLoading.IncrementalLoadingCollection<int> IncrementalLoadingSource { get; set; }

        public Class() 
        {
            IncrementalLoadingSource = new Windows.Collections.IncrementalLoading.IncrementalLoadingCollection<int>(GetPageItems, 5);
        }
    }
```

OR 

01.You can replace both 1, 2 steps by initialize IncrementalLoadingCollection using Lambda Expressions like following:

```C#
    public class Class
    {
        public Windows.Collections.IncrementalLoading.IncrementalLoadingCollection<int> IncrementalLoadingSource { get; set; }

        public Class() 
        {
            IncrementalLoadingSource = new Windows.Collections.IncrementalLoading.IncrementalLoadingCollection<int>(
                async (currentPage, pageSize) =>
                {
                    return Source.Skip(currentPage * pageSize).Take(pageSize);
                }, 5);
        }
    }
```

To supress the warning and run the method asynchronously (which is better for UI responsivness), you can wrap your code inside a Task block, it will look lik that:

```C#
    public class Class
    {
        public Windows.Collections.IncrementalLoading.IncrementalLoadingCollection<int> IncrementalLoadingSource { get; set; }

        public Class() 
        {
            IncrementalLoadingSource = new Windows.Collections.IncrementalLoading.Collections.IncrementalLoadingCollection<int>(
                async (currentPage, pageSize) =>
                {
                    return await System.Threading.Tasks.Task.Run<IEnumerable<int>>(() =>
                    {
                        return Source.Skip(currentPage * pageSize).Take(pageSize);
                    });
                }, 5);
        }
    }
```

03.Binding ListView control ItemsSource property to Incremental.PagedItems property Like That (After setting the DataContext).

```XAML
<ListView ItemsSource="{Binding IncrementalLoadingSource.PagedItems}"/>
```

Now the data bound to ListView are paged on ListView scrolling event.
