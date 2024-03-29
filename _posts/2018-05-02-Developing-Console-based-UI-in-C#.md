---
layout: post
title: Developing Console-based UI in C#
tags: C#, .NET Core
---
As you know Console Applications don't have graphical user interface or GUI. Instead, they run from Command Line, for example instead of writing a name into a textbox and clicking a button we would instead invoke the Console Application and provide the name as a parameter. In fact, in Console Applications we instead interact with the user using text inputs. For example, we could prompt something like this:

<img src="/public/img/consoleTest.jpg" width="700">

The user can then write a name and hit enter then we can read that text input into our application.

Wouldn't be better to make it beautiful? This is where we can use [Terminal.Gui](https://github.com/migueldeicaza/gui.cs) a cross-platform GUI toolkit:

<img src="/public/img/guics.png" alt="Sample app" width="700">

> Terminal.Gui is a library intended to create console-based applications using C#. The framework has been designed to make it easy to write applications that will work on monochrome terminals, as well as modern color terminals with mouse support. This library works across Windows, Linux and MacOS.


Using this UI toolkit is pretty simple, all you need to do is adding its package into your project:

```
dotnet add package Terminal.Gui
```

The simplest application looks like this:

{% highlight csharp %}
using System;

namespace consoleTest
{
    using Terminal.Gui;

    class Demo {
        static int Main ()
        {
            Application.Init ();

            var n = MessageBox.Query (50, 7,
                "Question", "Do you like console apps?", "Yes", "No");

            return n;
        }
    }
}

{% endhighlight %}

This example shows a prompt and returns an integer value depending on which value was selected by the user (Yes, No, or if they use chose not to make a decision and instead pressed the ESC key):

<img src="/public/img/firstTerminalUIapp.png" alt="Sample app" width="700">

As you can see the first thing to do is calling `Application.Init ();` to actually initialize the application. We can also create a window and then add a menu to it:

{% highlight csharp %}
Application.Init ();
var top = Application.Top;

// Creates the top-level window to show
var win = new Window (new Rect (0, 1, top.Frame.Width, top.Frame.Height-1), "MyApp");
top.Add (win);

// Creates a menubar, the item "New" has a help menu.
var menu = new MenuBar (new MenuBarItem [] {
    new MenuBarItem ("_File", new MenuItem [] {
        new MenuItem ("_New", "Creates new file", ()=> {}),
        new MenuItem ("_Close", "", () => {}),
        new MenuItem ("_Quit", "", () => { top.Running = false; })
    }),
    new MenuBarItem ("_Edit", new MenuItem [] {
        new MenuItem ("_Copy", "", null),
        new MenuItem ("C_ut", "", null),
        new MenuItem ("_Paste", "", null)
    })
});
top.Add (menu);

// Add some controls
win.Add (
    new Label (3, 2, "Login: "),
    new TextField (14, 2, 40, ""),
    new Label (3, 4, "Password: "),
    new TextField (14, 4, 40, "") {  },
    new CheckBox (3, 6, "Remember me"),
    new RadioGroup (3, 8, new [] { "_Personal", "_Company" }),
    new Button (3, 14, "Ok"),
    new Button (10, 14, "Cancel"),
    new Label (3, 18, "Press ESC and 9 to activate the menubar"));

Application.Run ();
{% endhighlight %}

<img src="/public/img/sampleTerminalUI.png" alt="Sample app" width="700">

You can build even complex console-based UI using this toolkit:

{% highlight csharp %}
using System;
using System.Linq;
using NStack;
using Terminal.Gui;
using TestingGuiCS.Models;

namespace TestingGuiCS
{
    class Program
    {
        static void Main(string[] args)
        {
            Application.Init ();
            var top = Application.Top;

            // Creates a menubar, the item "New" has a help menu.
            var menu = new MenuBar (new MenuBarItem [] {
                new MenuBarItem ("_File", new MenuItem [] {
                    new MenuItem ("_Quit", "", () => { top.Running = false; })
                })
            });
            top.Add (menu);


            // Creates the top-level window to show
            var win = new Window (new Rect (0, 1, top.Frame.Width, top.Frame.Height-1), "Movie Db");
            top.Add (win);

            // Add some controls
            var txtSearchLbl = new Label(3, 1, "Movie Name: ");
            var txtSearch = new TextField(15, 1, 30, "");
            var forKidsOnly = new CheckBox(3, 3, "For Kids?");
            var minimumRatingLbl = new Label(25, 3, "Minimum Rating: ");
            var minimumRatingTxt = new TextField(41, 3, 10, "");
            var searchBtn = new Button(3, 5, "Filter");
            var allMoviesListView = new ListView(new Rect(4, 8, top.Frame.Width, 200), MovieDataSource.GetList(forKidsOnly.Checked, 0).ToList());
            searchBtn.Clicked = () =>
            {

                double rating = 0;
                var isDouble = double.TryParse(minimumRatingTxt.Text.ToString(), out rating);
                if(!string.IsNullOrEmpty(minimumRatingTxt.Text.ToString()) && !isDouble)
                {
                    MessageBox.ErrorQuery(30, 6, "Error", "Rating must be number");
                    minimumRatingTxt.Text = ustring.Empty;
                    return;
                }

                win.Remove(allMoviesListView);
                if (string.IsNullOrEmpty(txtSearch.Text.ToString()) || string.IsNullOrEmpty(minimumRatingTxt.Text.ToString()))
                {
                    allMoviesListView = new ListView(new Rect(4, 8, top.Frame.Width, 200),
                        MovieDataSource.GetList(forKidsOnly.Checked, rating).ToList());
                    win.Add(allMoviesListView);
                }
                else
                {
                    win.Remove(allMoviesListView);
                    win.Add(new ListView(new Rect(4, 8, top.Frame.Width, 200),
                        MovieDataSource.GetList(forKidsOnly.Checked, rating)
                            .Where(x =>
                            x.Name.Contains(txtSearch.Text.ToString(), StringComparison.OrdinalIgnoreCase)
                        ).ToList()));
                }

            };
            win.Add (
                txtSearchLbl,
                txtSearch,
                forKidsOnly,
                minimumRatingLbl,
                minimumRatingTxt,
                searchBtn,
                new Label (3, 7, "-------------Search Result--------------")
            );

            Application.Run ();
        }
    }
}
{% endhighlight %}

<img src="/public/img/terminalUI.gif" alt="Sample app" width="700">


You can grab the working sample project from [GitHub](https://github.com/SirwanAfifi/Console-based-UI).

### What about cross platform UI frameworks?
Developing cross platform UIs is not officially supported by Microsoft, But there are some open source projects out there for doing so:
- [Avalonia](https://github.com/AvaloniaUI/Avalonia)
- [Eto.Forms](https://github.com/picoe/Eto)

