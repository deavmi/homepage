---
title: "Gustav: The graphical GTK client for DNET"
author: Tristan B. Kildaire
date: 2021-01-25
---

{{<bruh>}}
<div align="left">
<h3>Introduction<br>
</h3>
I spent the end of the year (the last few months) working on my
GTK DNET client and I thought I shoould share a progress update on
how that has gone so far. I should note that this is more of a
backlog blog in the sense that I haven't worked on the client
since a few months ago. So this won't be a really deep blog post
but it will just be a collection of screenshots and descriptions
of them just to show the progress I have been making with the
client.<br>
<br>
<div align="center"><img src="/img/gustav/gustav1.png" alt=""
width="960" height="585"></div>
</div>
<div align="center"> <br>
This is the first basic version of the client that I got working.
As you can see it would list all channels (not actually the ones
you joined) in the left-hand side sidebar, the right<br>
hand side would be the users in the currently selected channel.
Clicking on a channel would join you to the channel. Another thing
to mention is the notebook<br>
(the tabs at the top) - the client has support for multiple
sessions active at the same time.<br>
</div>
&nbsp;&nbsp;&nbsp; <br>
<br>
<div align="center"><img src="/img/gustav/gustav2.png" alt=""
width="1130" height="599"><br>
<br>
Here I am showing more messages coming through. As you can see it
was showing messages not yet decoded but rather represented as<br>
the bytes received. Some, like as with the above screenshot, were
being decoded by their message types into the leaves and joins. I
believe<br>
that at this stage I was logging all messages to one scrollable
text pane (to be correct a text pan in a GTK Box which is
scrollable) so it was<br>
not specific to the channel but rather ALL channels were being
showed, I would fix this in a later commit. At this stage I just
wanted a basic<br>
UI working with all the needed messages and notifications coming
through. The messages (text messages) are those ones right at the
bottom.<br>
<br>
<br>
<img src="/img/gustav/gustav3.png" alt="" width="1261" height="668"><br>
<br>
This is where stuff really started picking up speed in terms of
not only having a working client but rather one in which the
channel logs were separated.<br>
Here the title of the channel was added to the channel's view, as
you can see with the selected channel alan. Also clicking a
channel now would join you to it<br>
and the sidebar was used for that. What the notebook switcher (the
tabs at the top) represented was the channels you have already
joined and you could<br>
then easily switch between them like that. I also added a "Send"
button to allow you to type into a textbox (which you cannot see
here) and then hit send<br>
to send to the channel. So it was effectively up to speed with
what the terminal client, <i>skippy</i>, could do feature wise
now.<br>
<br>
I don't thing so maybe completely, the message decoding for
received messages wasn't there yet - I believe the message you see
there was me appending<br>
it to the message log after sending a test message.<br>
<br>
<br>
<img src="/img/gustav/gustav4.png" alt="" width="1242"
height="840"><br>
<br>
The client then got A LOT of stuff added to it (as you can now see
in the toolbar). There is now a toolbar which allows you<br>
to select what presence status you want set, Available, Away or
Busy (from left to right).<br>
<br>
<br>
<img src="/img/gustav/gustav5.png" alt="" width="428" height="331"><br>
<br>
I also add a little GTK about box expected of most GTK
applications<br>
<br>
<br>
<br>
<img src="/img/gustav/gustav6.png" alt="" width="428" height="331"><br>
<br>
Of course the credits screen also shows<br>
<br>
<br>
<img src="/img/gustav/gustav7.png" alt="" width="283" height="449"><br>
<br>
I then wanted to have a screen that listed channels that when you
clicked on them you<br>
would join the channel (as the sidebar now was becoming not a
total channels list but<br>
one that listed the channels YOU have explicitly joined.<br>
<br>
<br>
<img src="/img/gustav/gustav8.png" alt="" width="1040"
height="585"><br>
<br>
Here is where the presence messages can be seen with the black
tooltip<br>
that pops up to the left of your screen when you hover over a user
in the user's<br>
list sidebar<br>
<br>
<br>
<img src="/img/gustav/gustav9.png" alt="" width="1035"
height="548"><br>
<br>
Messages go to and fro, entitled with the username of the
message's author in bold<br>
<br>
<br>
<br>
<img src="/img/gustav/gustav11.png" alt="" width="1064"
height="597"><br>
<br>
It was ACTUALLY only here where the <i>channels</i> sidebar to
the left became a selector and list of channels<br>
you joined and not a list of channels you can click on to join,
however everything culminated to this and<br>
it is a good prototype graphical client for DNET. The image shown
is what multiple media would look<br>
like but this is not yet implemented - that read an image from
disk and rendered it. You can also see that<br>
there is now a join button for each channel in the channel list
and also each channel now shows a member<br>
count that is fetched when the window is opened.<br>
</div>
<br>
<br>
That is all for now but I will be posting more updates soon when I
get back to work on this project as some stuff has changed, more
addition-wise than anything you've seen here - that's stayed the
same.<br>
{{</bruh>}}