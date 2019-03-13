+++
title = "Building your app"
chapter = false
weight = 10
+++

We need to Create the ChatHub class. Create a new file called `ChatHub.cs` in the Hubs folder. Paste the following code into the file.

```
using System.Threading.Tasks;
using System.Collections.Generic;
using System;
using System.Linq;

namespace SignalRChat.Hubs
{
    public class ChatHub
    {
        static List<Tuple<int, int>> responses;
        static Tuple<string, string, string> currentQuestion;

        public ChatHub() {
            if (responses == null)
            {
                responses = new List<Tuple<int, int>>();
            }
        }
    }
}
```
Add the SignalR libray to the top of the file. 
```
using Microsoft.AspNetCore.SignalR;
```

Now change the ChatHub so that it inherits from the the class Hub which comes from the SignalR library.

```
public class ChatHub : Hub
```

Add this method to the Class. This method invoked every time a client connects. It sends whatever the current question is to the client. So if a client refreshes their browser or enters the quiz late, they will be sent the current question so that they can vote.

```
        public override async Task OnConnectedAsync()
        {
            await Clients.Caller.SendAsync("ReceiveCurrentQuestion", currentQuestion);
            await base.OnConnectedAsync();
        }
```

Navigate to `Startup.cs`
At the top of the file add a reference to the file we just created.

```
using SignalRChat.Hubs;
```

Inside `startup.cs` add the following line to the bottom of the `ConfigureServices` void. Around line 35

```
services.AddSignalR();
```

Next, we need to add a route where our hub will listen. It's this hub which will act as the endpoint to which the client side can connect. Add the following code to the bottom of the `public void Configure` method.

```
app.UseSignalR(routes =>
            {
                routes.MapHub<ChatHub>("/chatHub");
            });
```

Navigate to the view `Views/Home/Management.cshtml` add the following code into the `@section scripts {` portion

```
   <script>
        var app = new Vue({
            el: '#app',
            data: {
                messageList: [],
                question: '',
                option1: '',
                option2: '',
                option1value: '',
                option2value: '',
                active: false,
                percent: 50
            },
            methods: {
                
            },
            created: function () {
                this.connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();
            },
            mounted: function () {
                this.connection.start();
            }
        })
    </script>
```
Just below that closing script tag, add this next block of code. Basically when the Reveal.js system loads up the page will invoke the Start function on the server. 

Reveal.js is the UI system we use on the Management page. It's a library for building slide decks. My hope for this system is that I will be able to create interactive slide decks and have content interlaced with interactive quiz questions.

```
<script>
    Reveal.initialize();
    Reveal.addEventListener('start', function () {

        app.connection.invoke("Start").catch(function (err) {
            return console.error(err.toString());
        });
        event.preventDefault();
    });
</script>
```

Add this method to the class Hubs/ChatHub.cs
```
        public async Task Start()
        {
            ChatHub.currentQuestion = null;
            ChatHub.responses.Clear();
            await Clients.All.SendAsync("ReceiveCurrentQuestion", currentQuestion);
        }
```

If you run the app now. At the route of folder open a terminal and type:

```
dot net run
```

Once the site is running go to http://localhost:5000/home/management. You will simply see the first screen. What we would like to see here is a selection of question.

![Management](/images/management-1.png)


### Adding Questions

In the File `Controllers/HomeController.cs` find the action `IActionResult Management()

Replace the action with this code which gets a set of questions from a JSON file and loads them into the view:

```
        public IActionResult Management()
        {
            ViewData["Host"] = $"{Request.Scheme}://{Request.Host}";
            
            var rootFolder = Directory.GetCurrentDirectory();
            List<SiteQuestions> questions = JsonConvert.DeserializeObject<List<SiteQuestions>>(System.IO.File.ReadAllText(Path.Combine(rootFolder, "SiteQuestions.json")));

            // Pass the question to the view
            return View(questions);
        }
```

At the top of the View `Views/Home/Management.cshtml` add a model to the page of type `List<SiteQuestions>`

```
@{
    ViewData["Title"] = "Management";
    Layout = "Slide";
    @model List<SiteQuestions>
}
```

For each of the questions we are going to add a section of UI.

The following code is a template that will produce some UI for Each Question. Find the comment `@* Paste The Code Here *@` inside `Views/Home/Management.cshtml` and paste over it with the following code

```
@foreach (var question in Model)
{
<section data-state="@question.DataState">
    <div class="sl-block" data-block-type="text" style="width: 384px; left: 48px; top: 105px; height: auto;">
        <div class="sl-block-style" style="z-index: 10; transform: rotate(360deg);">
            <div class="sl-block-content" style="text-align: center; z-index: 10;">
                <h2>@question.Option1</h2>
            </div>
        </div>
    </div>
    <div class="sl-block" data-block-type="text" style="width: 384px; left: 528px; top: 105px; height: auto;">
        <div class="sl-block-content" style="text-align: center; z-index: 11;">
            <h2>@question.Option2</h2>
        </div>
    </div>
    
    <div class="sl-block" data-block-type="image" style="width: 477px; height: 266px; left: 11px; top: 242px; min-width: 4px; min-height: 4px;">
        <div class="sl-block-content" style="z-index: 12;">
            <img style="" data-natural-width="480" data-natural-height="268" data-lazy-loaded="" data-src="@question.Image1" />
        </div>
    </div>
    <div class="sl-block" data-block-type="image" style="width: 453px; height: 265px; left: 494px; top: 242px; min-width: 4px; min-height: 4px;">
        <div class="sl-block-style" style="z-index: 13; transform: rotate(360deg);">
            <div class="sl-block-content" style="z-index: 13;">
                <img style="" data-natural-width="500" data-natural-height="292" data-lazy-loaded="" data-src="@question.Image2" />
            </div>
        </div>
    </div>
    <div class="sl-block" data-block-type="text" style="height: auto; width: 200px; left: 390px; top: 123px;">
        <div class="sl-block-content" style="z-index: 14; text-align: center;">
            <p style="left:90px">VS</p>
            <div class="pie">
                <div class="pieright" v-bind:style="{ left: percent + '%' }"></div>
            </div>
        </div>
    </div>
    <div class="sl-block" data-block-type="text" style="height: auto; min-width: 30px; min-height: 30px; width: 96px; left: 192px; top: 179px;">
        <div class="sl-block-content" style="z-index: 16;">
            <p>{{ option1value }}</p>
        </div>
    </div>
    <div class="sl-block" data-block-type="text" style="height: auto; min-width: 30px; min-height: 30px; width: 96px; left: 673px; top: 179px;">
        <div class="sl-block-content" style="z-index: 18;">
            <p>{{ option2value }}</p>
        </div>
    </div>
</section>
}
```

If you stop the site `ctrl + c` and then run again, you will notice that the [management](http://localhost:5000/home/management) page now has a slide deck that can be navigated with the arrows to the bottom right of the screen or with the arrow keys.

```
dotnet run
```

As the Reveal.js slide show changes slides, a DataState change event is fired by Reveal.js.

In effect, the event gets invoked when a new question comes into view.

We are going to invoke the Server side method 'SendQuestion' and pass in the two options for the question.

On the server side, we will then  broadcast this message to all connected clients

At the bottom of the `Views/Home/Management.cshtml` page, just before the closing script tag that you added earlier, paste the following code.

```
    @foreach (var question in Model){
        <text>
        Reveal.addEventListener('@question.DataState', function () {
            app.percent = 50;
            app.connection.invoke("SendQuestion", '@question.Question', '@question.Option1', '@question.Option2').catch(function (err) {
                return console.error(err.toString());
            });
            event.preventDefault();
        });
        </text>
    }
```

Navigate to `Hubs/ChatHub.cs` and add the following method to the ChatHub class. This recives the message from the managment page when a new question is navigated to. It first makes the incoming question the currentQuestion and then broadcasts it to all connected clients by running `ReceiveCurrentQuestion` and passing the currentquestion. That will execute the JavaScript function on the client called `ReceiveCurrentQuestion` on the Client Side. You can take a look at this file and observe the JavaScript functions by navigating to `Views/Home/Index.cshtml`
```
       public async Task SendQuestion(string question, string option1, string option2)
        {
            ChatHub.currentQuestion = new Tuple<string,string,string>(question,option1,option2);
            // Reset all Answers
            ChatHub.responses.Clear();
           await Clients.All.SendAsync("ReceiveCurrentQuestion", currentQuestion);
        }
```

In the file `Views/Home/Index.cshtml` you might have noted that when a user presses a button to vote, it sends a response to a server side function called `SendResponse`. 

Add this method to the class `Hubs/ChatHub.cs`
```
        public async Task SendResponse(int one, int two)
        {
            ChatHub.responses.Add(new Tuple<int,int>(one,two));
            var option1count = ChatHub.responses.Sum(t => t.Item1);
            var option2count = ChatHub.responses.Sum(t => t.Item2);
            await Clients.All.SendAsync("ReceiveResponses", option1count, option2count);
        }
```

In turn this function calls a function called `ReceiveResponses` on the management page.

In `Views/Home/Management.cshtml` find the JavaScript function `mounted: function () {`
add these two functions that will be called by the CSharp ChatHub when the question changes and when people vote.

```
this.connection.on("ReceiveCurrentQuestion", (question, message) => {
    if (question !== null && question !== undefined && question.item1 != '') {
        this.option1value = "";
        this.option2value = "";
    }
});

this.connection.on("ReceiveResponses", (one, two) => {

    // Update Number
    this.option1value = one;
    this.option2value = two;

    // Update Graph
    var number1 = one;
    var number2 = one + two;
    this.percent = Math.floor((number1 / number2) * 100);
    
});
```

### Test The Site

If you stop the site `ctrl + c` and then run again

```
dotnet run
```

1. Open up the Management Page: http://localhost:5000/home/management
2. Open up the Index Page: http://localhost:5000/

As you navigate around the slides you should notice that the index page question changes.

If you vote, you should notice that the graph on the slide updates.

{{% notice tip %}}
If you refresh the user page you can vote again. There is no logic to prevent this from happening. This is a good way to see how the votes change the slides and the graph.
{{% /notice %}}


