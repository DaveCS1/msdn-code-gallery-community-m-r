# Next Steps : Azure Service Bus Messaging with Queues using Sessions
## Requires
- Visual Studio 2013
## License
- MIT
## Technologies
- C#
- Microsoft Azure
- Service Bus
- Visual C#
- .NET 4.5
## Topics
- C#
- Azure
- Microsoft Azure
- Service Bus
## Updated
- 10/06/2015
## Description

<h1>Introduction</h1>
<p>This sample project builds upon the Azure SDK Template <em>Azure Service Bus: Messaging With Queues</em> by adding sessions and using the dead letter queue.</p>
<h1>Building the Project</h1>
<p>The project should download all required packages when built. &nbsp;You will first need to update the endpoint definition in the app.config of the project with a valid endpoint.</p>
<p>You can use the <a href="https://azure.microsoft.com/en-us/documentation/articles/service-bus-dotnet-how-to-use-queues/">
How to use Azure Service Bus queues</a>&nbsp;article to determine a suitable endpoint.</p>
<p>The following links are provided as an introduction:</p>
<p><em><a href="https://code.msdn.microsoft.com/Getting-Started-Brokered-aa7a0ac3">Getting Started: Messaging with Queues</a></em></p>
<p><em><a href="http://blogs.msdn.com/b/brunoterkaly/archive/2014/08/07/learn-how-to-create-a-queue-place-and-read-a-message-using-azure-service-bus-queues-in-5-minutes.aspx">Learn how to create a queue, place and read a message using Azure Service Bus Queues
 in 5 minutes</a></em></p>
<h1>Background</h1>
<p><span>In scenarios where there are multiple Azure Service Bus clients and you want to route particular messages to particular consumers, sessions are an ideal feature of the Azure Service Bus Queue. &nbsp;</span></p>
<p><span>&nbsp;</span>There are many scenarios where this could be useful. Here are three examples:</p>
<ul>
<li><strong>Priority -</strong>&nbsp;A low priority queue and a high priority queue could be created where the high priority items are worked before the low priority queue items.
</li><li><strong>Partitioning -</strong>&nbsp;The distribution of the queue items across multiple message brokers could be used to distribute the load.
</li><li><strong>Aggregation -</strong>&nbsp;Routing messages to specific message brokers could be used if specific messages need to be sent to specified consumers.&nbsp;
</li></ul>
<h1>Description</h1>
<p>To illustrate sessions, three messages will be posted to the service bus. Two will have the same session id and the third will have a different session id.</p>
<p>We will then use two different approches to consuming the messages. &nbsp;The first will be to create two clients to consume the messages specifying a particularly session id. &nbsp;The second will be to use an implemenation of IMessageSessionHandler.</p>
<h2>Queue Description</h2>
<p>The queue must be created to require sessions. &nbsp;This is done as follows:</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp">var&nbsp;description&nbsp;=&nbsp;<span class="cs__keyword">new</span>&nbsp;QueueDescription(QueueName)&nbsp;
{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;RequiresSession&nbsp;=&nbsp;<span class="cs__keyword">true</span>&nbsp;
};&nbsp;
&nbsp;
namespaceManager.CreateQueue(description);</pre>
</div>
</div>
</div>
<h2 class="endscriptcode">BrokeredMessage&nbsp;</h2>
<div class="endscriptcode">When a message created, a session must be specified otherwise the send will result in an exception. &nbsp;This can be done when the message is constructed:</div>
<div class="endscriptcode">
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js">BrokeredMessage&nbsp;message&nbsp;=&nbsp;<span class="js__operator">new</span>&nbsp;BrokeredMessage(messageBody)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MessageId&nbsp;=&nbsp;messageId,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SessionId&nbsp;=&nbsp;sessionId&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>;</pre>
</div>
</div>
</div>
<h2 class="endscriptcode">QueueClient</h2>
</div>
<p>The first approach to consuming the messages will be to use a QueueClient object. &nbsp;As our application is the only consumer, I decided to use a foreach statement to loop through all the sessions and consume all the messages in each sessions. &nbsp;</p>
<p>Note: <em>to illustrate DeadLetter, I am sending the items from Session2 to DeadLetter</em>.</p>
<p>The following is the modified method for consuming the items:</p>
<p>&nbsp;</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js">private&nbsp;static&nbsp;<span class="js__operator">void</span>&nbsp;ReceiveMessages()&nbsp;
<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Console.WriteLine(<span class="js__string">&quot;\nReceiving&nbsp;message&nbsp;from&nbsp;Queue...&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;BrokeredMessage&nbsp;message&nbsp;=&nbsp;null;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">var</span>&nbsp;sessions&nbsp;=&nbsp;queueClient.GetMessageSessions();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;foreach&nbsp;(<span class="js__statement">var</span>&nbsp;browser&nbsp;<span class="js__operator">in</span>&nbsp;sessions)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Console.WriteLine(string.Format(<span class="js__string">&quot;Session&nbsp;discovered:&nbsp;Id&nbsp;=&nbsp;{0}&quot;</span>,&nbsp;browser.SessionId));&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">var</span>&nbsp;session&nbsp;=&nbsp;queueClient.AcceptMessageSession(browser.SessionId);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">while</span>&nbsp;(true)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">try</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;message&nbsp;=&nbsp;session.Receive(TimeSpan.FromSeconds(<span class="js__num">5</span>));&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">if</span>&nbsp;(message&nbsp;!=&nbsp;null)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Console.WriteLine(string.Format(<span class="js__string">&quot;Message&nbsp;received:&nbsp;Id&nbsp;=&nbsp;{0},&nbsp;Body&nbsp;=&nbsp;{1}&quot;</span>,&nbsp;message.MessageId,&nbsp;message.GetBody&lt;string&gt;()));&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">if</span>&nbsp;(session.SessionId&nbsp;==&nbsp;SessionId2)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__sl_comment">//&nbsp;if&nbsp;this&nbsp;is&nbsp;the&nbsp;second&nbsp;session&nbsp;then&nbsp;let's&nbsp;send&nbsp;to&nbsp;the&nbsp;dead&nbsp;letter&nbsp;for&nbsp;fun</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;message.DeadLetter();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Further&nbsp;custom&nbsp;message&nbsp;processing&nbsp;could&nbsp;go&nbsp;here...&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;message.Complete();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">else</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__sl_comment">//no&nbsp;more&nbsp;messages&nbsp;in&nbsp;the&nbsp;queue</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">break</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">catch</span>&nbsp;(MessagingException&nbsp;e)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">if</span>&nbsp;(!e.IsTransient)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Console.WriteLine(e.Message);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">throw</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">else</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HandleTransientErrors(e);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">}</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;queueClient.Close();&nbsp;
<span class="js__brace">}</span></pre>
</div>
</div>
</div>
<div class="endscriptcode"></div>
<p>&nbsp;</p>
<h2>IMessageSessionHandler</h2>
<p>Using IMessageSessionHandler is a great way to consume the messages if the session id is not known. &nbsp;You first register the session handler against the queue:</p>
<p>&nbsp;</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js">queueClient.RegisterSessionHandler(<span class="js__operator">typeof</span>(MyMessageSessionHandler),&nbsp;<span class="js__operator">new</span>&nbsp;SessionHandlerOptions&nbsp;<span class="js__brace">{</span>&nbsp;AutoComplete&nbsp;=&nbsp;false&nbsp;<span class="js__brace">}</span>);</pre>
</div>
</div>
</div>
<div class="endscriptcode">Messages are then received by the implementation in the OnMessage method:</div>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js">public&nbsp;<span class="js__operator">void</span>&nbsp;OnMessage(MessageSession&nbsp;session,&nbsp;BrokeredMessage&nbsp;message)&nbsp;
<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Console.WriteLine(string.Format(<span class="js__string">&quot;MyMessageSessionHandler&nbsp;{3}&nbsp;received&nbsp;messaged&nbsp;on&nbsp;session&nbsp;Id&nbsp;=&nbsp;{0},&nbsp;Id&nbsp;=&nbsp;{1},&nbsp;Body&nbsp;=&nbsp;{2}&quot;</span>,&nbsp;session.SessionId,&nbsp;message.MessageId,&nbsp;message.GetBody&lt;string&gt;(),&nbsp;WhoAmI));&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;message.Complete();&nbsp;
<span class="js__brace">}</span></pre>
</div>
</div>
</div>
<h2 class="endscriptcode">DeadLetter</h2>
<div class="endscriptcode">There are several uses for the DeadLetter channel on a queue but it is primarily used when messages failed to be handled or expired before they were completed.</div>
<div class="endscriptcode"></div>
<div class="endscriptcode">In the project, items are explicitly sent to DeadLetter:</div>
<div class="endscriptcode">
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js">message.DeadLetter();</pre>
</div>
</div>
</div>
<div class="endscriptcode">And likewise read from the DeadLetter channel:</div>
<div class="endscriptcode">&nbsp;</div>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="js"><span class="js__statement">var</span>&nbsp;deadLetterPath&nbsp;=&nbsp;QueueClient.FormatDeadLetterPath(QueueName);&nbsp;
&nbsp;
<span class="js__statement">var</span>&nbsp;deadLetterClient&nbsp;=&nbsp;QueueClient.Create(deadLetterPath);&nbsp;
&nbsp;
<span class="js__statement">while</span>&nbsp;(true)&nbsp;
<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__statement">try</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="js__brace">{</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;message&nbsp;=&nbsp;deadLetterClient.Receive(TimeSpan.FromSeconds(<span class="js__num">5</span>));</pre>
</div>
</div>
</div>
<h1 class="endscriptcode">Summary</h1>
<p>The Azure Service Bus offers a huge amount of scope that might be overlooked initially due to its overall simplicity when getting started. &nbsp;Sessions and DeadLetter are just two features that extend the capability of the service bus.</p>
<p>Any feedback is appreciated!</p>
</div>
<p>&nbsp;</p>
