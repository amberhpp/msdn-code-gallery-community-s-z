# Using MongoDB with ASP.NET Web API
## Requires
- Visual Studio 2010
## License
- MS-LPL
## Technologies
- ASP.NET
- .NET Framework
- ASP.NET Web API
- HttpClient
## Topics
- REST
- HTTP
- Web API
## Updated
- 09/01/2012
## Description

<div><span style="font-size:medium"><strong><span style="background-color:#ffff00">This code is no longer maintained. Please see
<a href="http://blogs.msdn.com/b/webdev/archive/2012/08/26/asp-net-web-api-and-httpclient-samples.aspx">
List of ASP.NET Web API and HttpClient Samples</a> for updated samples.</span></strong></span></div>
<div><span style="font-size:medium"><strong><span style="background-color:#ffff00"><br>
</span></strong></span></div>
<div></div>
<div><a href="http://www.mongodb.org/">MongoDB</a> is a popular <a href="http://en.wikipedia.org/wiki/NoSQL">
NoSQL database</a> that makes it a great backend for Web APIs which lend themselves towards a
<a href="http://en.wikipedia.org/wiki/Document-oriented_database">document store</a> rather than a
<a href="http://en.wikipedia.org/wiki/Relational_database">relational store</a>. In this blog we show how you can use
<a href="http://www.mongodb.org/">MongoDB</a> with <a href="http://www.asp.net/web-api">
ASP.NET Web API</a> to build an ApiController with support for HTTP GET, PUT, POST, and DELETE.</div>
<p>Please see the blog <a href="http://blogs.msdn.com/b/henrikn/archive/2012/02/19/using-web-api-with-mongodb.aspx">
&quot;Using MongoDB with ASP.NET Web API&quot;</a> for details.</p>
<p>Our sample controller is a simple contact manager Web API which builds on many of the same concepts as in the tutorial &ldquo;<a href="http://www.asp.net/web-api/overview/web-api-routing-and-actions/creating-a-web-api-that-supports-crud-operations">Creating
 a Web API that Supports CRUD Operations</a>&rdquo; so I would recommend that you skim that for background.</p>
<p>In particular, the URI patterns are very similar to that of the tutorial:</p>
<table border="1" cellspacing="0" cellpadding="2" width="500" align="center">
<tbody>
<tr>
<td width="133" valign="top"><strong>Action</strong></td>
<td width="133" valign="top"><strong>HTTP Method</strong></td>
<td width="133" valign="top"><strong>Relative URI</strong></td>
</tr>
<tr>
<td width="133" valign="top">Get a list of all contacts</td>
<td width="133" valign="top">GET</td>
<td width="133" valign="top">/api/contacts</td>
</tr>
<tr>
<td width="133" valign="top">Get a filtered list of all contacts</td>
<td width="133" valign="top">GET</td>
<td width="133" valign="top">/api/contacts?$top=2</td>
</tr>
<tr>
<td width="133" valign="top">Get a contacts by ID</td>
<td width="133" valign="top">GET</td>
<td width="133" valign="top">/api/contacts/id</td>
</tr>
<tr>
<td width="133" valign="top">Create a new contact</td>
<td width="133" valign="top">POST</td>
<td width="133" valign="top">/api/contacts</td>
</tr>
<tr>
<td width="133" valign="top">Update a contact</td>
<td width="133" valign="top">PUT</td>
<td width="133" valign="top">/api/contacts/id</td>
</tr>
<tr>
<td width="133" valign="top">Delete a contact</td>
<td width="133" valign="top">DELETE</td>
<td width="133" valign="top">/api/contacts/id</td>
</tr>
</tbody>
</table>
<p>However, there are two differences worth noting:</p>
<ol>
<li>We host the sample ApiController in selfhost with a base URI of <a href="http://localhost:8080">
http://localhost:8080</a> </li><li>We have added support for <a href="http://www.odata.org/developers/protocols/uri-conventions#QueryStringOptions">
OData-style queries</a> on GET requests so that you can filter the set of contacts returned using $top, $skip, etc.
</li></ol>
<p>But I am getting ahead of myself &ndash; let&rsquo;s get back to building the controller&hellip;</p>
<h1>Prerequisites</h1>
<p>Before you start you obviously need to download and install <a href="http://www.mongodb.org/">
MongoDB</a>. I followed the <a href="http://www.mongodb.org/display/DOCS/Quickstart&#43;Windows">
quick start instructions</a> which has all the information you need and got me going in 10 minutes or so.</p>
<p>Second you need the MongoDB driver for C#. Here we use the <a href="http://www.mongodb.org/display/DOCS/CSharp&#43;Language&#43;Center">
MongoDB C# driver</a> provided by <a href="https://www.10gen.com/">10gen</a> and which is available as a
<a href="http://nuget.org/packages/mongocsharpdriver">NuGet package</a>.</p>
<p>As an optional third step once your MongoDB installation has been set up you can use a tool like
<a href="http://www.mongovue.com/MongoVUE">MongoVUE</a> which allows you to browse and interact directly with the data in a MongoDB database.</p>
<h1>Defining the Contact Type</h1>
<p>We use the same basic Contact type used in &ldquo;<a href="http://www.asp.net/web-api/overview/web-api-routing-and-actions/creating-a-web-api-that-supports-crud-operations">Creating a Web API that Supports CRUD Operations</a>&rdquo; but decorate it with
 an BsonIdAttribute (see the <a href="http://api.mongodb.org/csharp/1.0/index.html">
Mongo C# driver reference documentation</a> for details) indicating which field is the id:</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp"><span class="cs__keyword">public</span><span class="cs__keyword">class</span>&nbsp;Contact&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[BsonId]&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span><span class="cs__keyword">string</span>&nbsp;Id&nbsp;{&nbsp;<span class="cs__keyword">get</span>;&nbsp;<span class="cs__keyword">set</span>;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span><span class="cs__keyword">string</span>&nbsp;Name&nbsp;{&nbsp;<span class="cs__keyword">get</span>;&nbsp;<span class="cs__keyword">set</span>;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span><span class="cs__keyword">string</span>&nbsp;Phone&nbsp;{&nbsp;<span class="cs__keyword">get</span>;&nbsp;<span class="cs__keyword">set</span>;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span><span class="cs__keyword">string</span>&nbsp;Email&nbsp;{&nbsp;<span class="cs__keyword">get</span>;&nbsp;<span class="cs__keyword">set</span>;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;DateTime&nbsp;LastModified&nbsp;{&nbsp;<span class="cs__keyword">get</span>;&nbsp;<span class="cs__keyword">set</span>;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}</pre>
</div>
</div>
</div>
<h1>Defining the Contact Repository</h1>
<p>We use the common <a href="http://msdn.microsoft.com/en-us/library/ff649690.aspx">
repository pattern</a> to interact with the backend data source so the next step is to define the shape of the repository. We do this by defining an IContactRepository interface which we then implement so that it targets MongoDB. First, the interface is defined
 like this:</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp"><span class="cs__keyword">public</span><span class="cs__keyword">interface</span>&nbsp;IContactRepository&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IEnumerable&lt;Contact&gt;&nbsp;GetAllContacts();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Contact&nbsp;GetContact(<span class="cs__keyword">string</span>&nbsp;id);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Contact&nbsp;AddContact(Contact&nbsp;item);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">bool</span>&nbsp;RemoveContact(<span class="cs__keyword">string</span>&nbsp;id);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">bool</span>&nbsp;UpdateContact(<span class="cs__keyword">string</span>&nbsp;id,&nbsp;Contact&nbsp;item);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}</pre>
</div>
</div>
</div>
<p>&nbsp;</p>
<h1>Implementing the Repository</h1>
<p>It&rsquo;s when implementing IContactRepository that we use the <a href="http://api.mongodb.org/csharp/1.0/index.html">
MongoDB C# driver API</a> to connect to the data and to do the various CRUD operations defined by IContactRepository. First we set up the connection to database and get a Contact collection. By default we use the connection string &ldquo;mongodb://localhost:27017&rdquo;
 pointing at the localhost database we set up above.</p>
<p>For the purpose of this sample we also reset the database and add some default entries so we have something to start with. Note that
<a href="http://www.mongodb.org/display/DOCS/CSharp&#43;Driver&#43;Tutorial#CSharpDriverTutorial-Threadsafety">
the classes MongoServer, MongoDatabase, and MongoCollection&lt;T&gt; are all thread-safe</a> so they can be used simultaneously by separate threads.</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;ContactRepository(<span class="cs__keyword">string</span>&nbsp;connection)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>&nbsp;(<span class="cs__keyword">string</span>.IsNullOrWhiteSpace(connection))&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;connection&nbsp;=&nbsp;<span class="cs__string">&quot;mongodb://localhost:27017&quot;</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_server&nbsp;=&nbsp;MongoServer.Create(connection);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_database&nbsp;=&nbsp;_server.GetDatabase(<span class="cs__string">&quot;Contacts&quot;</span>,&nbsp;SafeMode.True);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_contacts&nbsp;=&nbsp;_database.GetCollection&lt;Contact&gt;(<span class="cs__string">&quot;contacts&quot;</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//&nbsp;Reset&nbsp;database&nbsp;and&nbsp;add&nbsp;some&nbsp;default&nbsp;entries</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_contacts.RemoveAll();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">for</span>&nbsp;(<span class="cs__keyword">int</span>&nbsp;index&nbsp;=&nbsp;<span class="cs__number">1</span>;&nbsp;index&nbsp;&lt;&nbsp;<span class="cs__number">5</span>;&nbsp;index&#43;&#43;)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Contact&nbsp;contact1&nbsp;=&nbsp;<span class="cs__keyword">new</span>&nbsp;Contact&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Email&nbsp;=&nbsp;<span class="cs__keyword">string</span>.Format(<span class="cs__string">&quot;test{0}@example.com&quot;</span>,&nbsp;index),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Name&nbsp;=&nbsp;<span class="cs__keyword">string</span>.Format(<span class="cs__string">&quot;test{0}&quot;</span>,&nbsp;index),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Phone&nbsp;=&nbsp;<span class="cs__keyword">string</span>.Format(<span class="cs__string">&quot;{0}{0}{0}&nbsp;{0}{0}{0}&nbsp;{0}{0}{0}{0}&quot;</span>,&nbsp;index)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AddContact(contact1);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}</pre>
</div>
</div>
</div>
<div class="endscriptcode">&nbsp;We now add the actual CRUD implementations as follows:</div>
<div class="endscriptcode"></div>
<div class="endscriptcode">
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;IEnumerable&lt;Contact&gt;&nbsp;GetAllContacts()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;_contacts.FindAll();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;Contact&nbsp;GetContact(<span class="cs__keyword">string</span>&nbsp;id)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IMongoQuery&nbsp;query&nbsp;=&nbsp;Query.EQ(<span class="cs__string">&quot;_id&quot;</span>,&nbsp;id);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;_contacts.Find(query).FirstOrDefault();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;Contact&nbsp;AddContact(Contact&nbsp;item)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;item.Id&nbsp;=&nbsp;ObjectId.GenerateNewId().ToString();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;item.LastModified&nbsp;=&nbsp;DateTime.UtcNow;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_contacts.Insert(item);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;item;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;<span class="cs__keyword">bool</span>&nbsp;RemoveContact(<span class="cs__keyword">string</span>&nbsp;id)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IMongoQuery&nbsp;query&nbsp;=&nbsp;Query.EQ(<span class="cs__string">&quot;_id&quot;</span>,&nbsp;id);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SafeModeResult&nbsp;result&nbsp;=&nbsp;_contacts.Remove(query);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;result.DocumentsAffected&nbsp;==&nbsp;<span class="cs__number">1</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;<span class="cs__keyword">bool</span>&nbsp;UpdateContact(<span class="cs__keyword">string</span>&nbsp;id,&nbsp;Contact&nbsp;item)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IMongoQuery&nbsp;query&nbsp;=&nbsp;Query.EQ(<span class="cs__string">&quot;_id&quot;</span>,&nbsp;id);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;item.LastModified&nbsp;=&nbsp;DateTime.UtcNow;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IMongoUpdate&nbsp;update&nbsp;=&nbsp;Update&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Set(<span class="cs__string">&quot;Email&quot;</span>,&nbsp;item.Email)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Set(<span class="cs__string">&quot;LastModified&quot;</span>,&nbsp;DateTime.UtcNow)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Set(<span class="cs__string">&quot;Name&quot;</span>,&nbsp;item.Name)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.Set(<span class="cs__string">&quot;Phone&quot;</span>,&nbsp;item.Phone);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SafeModeResult&nbsp;result&nbsp;=&nbsp;_contacts.Update(query,&nbsp;update);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;result.UpdatedExisting;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
</pre>
</div>
</div>
</div>
<div class="endscriptcode">
<h1>Creating the Contact Controller</h1>
<p>That&rsquo;s it for the repository so we can now implement the actual ApiController. We base the implementation on the common pattern for supporting GET, PUT, POST, and DELETE as follows</p>
</div>
</div>
<div class="endscriptcode">
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp">&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;<span class="cs__keyword">class</span>&nbsp;ContactsController&nbsp;:&nbsp;ApiController&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">private</span>&nbsp;<span class="cs__keyword">static</span>&nbsp;<span class="cs__keyword">readonly</span>&nbsp;IContactRepository&nbsp;_contacts&nbsp;=&nbsp;<span class="cs__keyword">new</span>&nbsp;ContactRepository();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;IQueryable&lt;Contact&gt;&nbsp;Get()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;_contacts.GetAllContacts().AsQueryable();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;Contact&nbsp;Get(<span class="cs__keyword">string</span>&nbsp;id)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Contact&nbsp;contact&nbsp;=&nbsp;_contacts.GetContact(id);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>&nbsp;(contact&nbsp;==&nbsp;<span class="cs__keyword">null</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">throw</span>&nbsp;<span class="cs__keyword">new</span>&nbsp;HttpResponseException(HttpStatusCode.NotFound);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;contact;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;Contact&nbsp;Post(Contact&nbsp;<span class="cs__keyword">value</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Contact&nbsp;contact&nbsp;=&nbsp;_contacts.AddContact(<span class="cs__keyword">value</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">return</span>&nbsp;contact;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;<span class="cs__keyword">void</span>&nbsp;Put(<span class="cs__keyword">string</span>&nbsp;id,&nbsp;Contact&nbsp;<span class="cs__keyword">value</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>&nbsp;(!_contacts.UpdateContact(id,&nbsp;<span class="cs__keyword">value</span>))&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">throw</span>&nbsp;<span class="cs__keyword">new</span>&nbsp;HttpResponseException(HttpStatusCode.NotFound);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">public</span>&nbsp;<span class="cs__keyword">void</span>&nbsp;Delete(<span class="cs__keyword">string</span>&nbsp;id)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>&nbsp;(!_contacts.RemoveContact(id))&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">throw</span>&nbsp;<span class="cs__keyword">new</span>&nbsp;HttpResponseException(HttpStatusCode.NotFound);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}</pre>
</div>
</div>
</div>
<div class="endscriptcode">
<p><strong>Note:</strong> We use IQueryable&lt;Contact&gt; as the return type of the Get() method. This enables automatic query-support using OData query syntax including
<strong>$top</strong> and <strong>$skip</strong>. That is, if you use a URI like &ldquo;/api/contacts?$top=2&rdquo; with a &ldquo;?$top=2&rdquo; query component then you will only get the first two entries back.</p>
<h1>Hosting the Controller</h1>
<p>Now that we have the controller we can either host it in ASP or as selfhost. Here we use selfhost to host the controller in a simple console application but it would work exactly the same if hosted in ASP. As usual we create a HttpSelfHostConfiguration,
 add a route, then create a server, and start it:</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>C#</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">csharp</span>

<div class="preview">
<pre class="csharp"><span class="cs__keyword">static</span><span class="cs__keyword">void</span>&nbsp;Main(<span class="cs__keyword">string</span>[]&nbsp;args)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HttpSelfHostServer&nbsp;server&nbsp;=&nbsp;<span class="cs__keyword">null</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">try</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//&nbsp;Set&nbsp;up&nbsp;server&nbsp;configuration</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HttpSelfHostConfiguration&nbsp;config&nbsp;=&nbsp;<span class="cs__keyword">new</span>&nbsp;HttpSelfHostConfiguration(<span class="cs__string">&quot;http://localhost:8080&quot;</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;config.Routes.MapHttpRoute(&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;<span class="cs__string">&quot;DefaultApi&quot;</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;routeTemplate:&nbsp;<span class="cs__string">&quot;api/{controller}/{id}&quot;</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;defaults:&nbsp;<span class="cs__keyword">new</span>&nbsp;{&nbsp;id&nbsp;=&nbsp;RouteParameter.Optional&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//&nbsp;Create&nbsp;server</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;=&nbsp;<span class="cs__keyword">new</span>&nbsp;HttpSelfHostServer(config);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//&nbsp;Start&nbsp;listening</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server.OpenAsync().Wait();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//&nbsp;Run&nbsp;HttpClient&nbsp;issuing&nbsp;requests</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;RunClient();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Console.WriteLine(<span class="cs__string">&quot;Hit&nbsp;ENTER&nbsp;to&nbsp;exit...&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Console.ReadLine();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">finally</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__keyword">if</span>&nbsp;(server&nbsp;!=&nbsp;<span class="cs__keyword">null</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="cs__com">//&nbsp;Stop&nbsp;listening</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server.CloseAsync().Wait();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
</pre>
</div>
</div>
</div>
<p>&nbsp;</p>
<p><strong>Note</strong>: In order to successfully start the selfhost server you have to run as admin (or
<a href="http://msdn.microsoft.com/library/windows/desktop/cc307236.aspx">configure http.sys with the appropriate URI prefix using netsh</a>).</p>
<h1>Running the Controller</h1>
<p>When running the controller can be accessed using any HTTP client. In the&nbsp;code we&nbsp;show how to use HttpClient to do GET and POST but
<a href="http://www.fiddler2.com">Fiddler</a> is often very useful for trying out various combinations of GET, PUT, POST, and DELETE by using the Composer tab manually to create requests. For example, you can create a POST request like this to insert a new
 Contact and then hit <strong>Execute</strong>:</p>
<p><a href="http://blogs.msdn.com/cfs-file.ashx/__key/communityserver-blogs-components-weblogfiles/00-00-01-50-67-metablogapi/3108.AddContact_5F00_5E44CF94.png"><img title="AddContact" src="http://blogs.msdn.com/cfs-file.ashx/__key/communityserver-blogs-components-weblogfiles/00-00-01-50-67-metablogapi/4505.AddContact_5F00_thumb_5F00_0FD03D2A.png" border="0" alt="AddContact" width="604" height="444" style="margin-right:auto; margin-left:auto; float:none; display:block"></a></p>
<p>Similarly you can create a GET request to ask for the two first entries which will yield a result like this where the response body contains the first two Contacts:</p>
<p><a href="http://blogs.msdn.com/cfs-file.ashx/__key/communityserver-blogs-components-weblogfiles/00-00-01-50-67-metablogapi/6330.GetTopContactsResult_5F00_1D365030.png"><img title="GetTopContactsResult" src="http://blogs.msdn.com/cfs-file.ashx/__key/communityserver-blogs-components-weblogfiles/00-00-01-50-67-metablogapi/5353.GetTopContactsResult_5F00_thumb_5F00_67BD8E0A.png" border="0" alt="GetTopContactsResult" width="604" height="459" style="margin-right:auto; margin-left:auto; float:none; display:block"></a></p>
<p>Have fun!</p>
<p>Henrik</p>
</div>
</div>
