---
title: Super simple AJAX forms without jQuery
date: 2015-05-20 14:19:31 Z
categories:
- ajax
layout: post
description: Formspree + a few lines of JavaScript = great looking forms for static
  sites.
image: email-forms
---

Formspree is a great solution for adding forms to static sites. It's super simple, it's free and it works.

At its most simple it's just a case of setting your form action so that your data gets POST-ed to Formspree. Here's the basic example given on the FormSpree site:

{% highlight html %}             
<form action="//formspree.io/your@email.com" method="POST">
	<input type="text" name="name" />
	<input type="email" name="_replyto" />
	<input type="submit" value="Send" />
</form>
{% endhighlight %}

  <!--  <form action="//formspree.io/john@jdp.org.uk" method="POST">
        <input type="text" name="Name" placeholder="Your name" />
        <input type="email" name="_replyto" placeholder="Your email" />
        <label><input type="radio" name="Gender" value="male" checked>Male</label>
        <label><input type="radio" name="Gender" value="female">Female</label>
        <textarea rows="2" name="Mesage"></textarea> 
        <label>How did you hear about us?
            <select name="source">
                <option value="Word of mouth">Word of mouth</option>
                <option value="Newspaper">Newspaper</option>
                <option value="Other">Other</option>
            </select>
        </label>
        <input type="submit" value="Send">
    </form> -->
Add this to your page and change your@email.com to (yes) your email. Then send a test submission which will result in a confirmation email being sent to prevent spam. Once you confirm this you're good to go.

You can add fields to this basic example to you heart's content. Each field will appear in the resulting emails like so:

![Screenshot showing how forms correlate to emails](/images/blog/email-forms-screenshot.jpg)

### Controlling what happens upon submission

There are three options covered on the Formspree site:


- Send the user to the [default thanks page](http://formspree.io/thanks)
- Send the user to an alternate URL
- Use AJAX (and essentially do whatever you like)


#### Send the user to the default thanks page
This is the default option and what will happen if you use the example code given above. It's not the worst solution in the world, but a) it provides an inconsistent experience for the user in terms of branding and b) the thanks page is a deadend – users are going to have to hit 'back' if they're to return to your site.

#### Provide an alternate URL

Adding a hidden field with the name '_next' lets us send the user where we want:

{% highlight html %}
<input type="hidden" name="_next" value="//site.io/thanks.html" />
{% endhighlight %}

This allows you to use an acknowledgement page within your own site, a nice step-up in consistency. In instances where the form is the main focus of the page it'll probably feel like the *right* solution, but when the form appers in a sidebar or an overlay, loading a new page will still feel heavyhanded and interupt the user's experience.

#### Use AJAX to make the submission

The advantage here is that the form submission and response is handled by Javascript, which gives us plenty of flexibility – including the ability to not interrupt the user's browsing.

##### With jQuery

Using AJAX is referenced on the Formspree site, but the example given is only really included to highlight the need to set `dataType:"json"` to avoid running into [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) issues:

{% highlight javascript %}
$.ajax({
    url: '//formspree.io/you@email.com', 
    method: 'POST',
    data: {message: 'hello!'},
    dataType: 'json'
});
{% endhighlight %}

Clearly there are some missing steps here, which shouldn't be too difficult to spot:

- The form data needs serialising into a JSON string
- We need to get the response from the server and do something – e.g. show a message depending on success/failure

Following these steps we end up with something like:

{% highlight javascript %}
$.ajax({
  url: 'http://formspree.io/email@address',
  method: 'POST',
  data: $('#myform').serialize(),
  dataType: 'json',
  beforeSend: function() { /* sending message... */ },
  success: function(data) { /* success message... */ },
  error: function(err) { /* error message... */ }
});
{% endhighlight %}

##### Without jQuery

When I came to implement a Formspree form on a Jekyll blog, I reached the step above but had already decided not to use jQuery on the site.

The advantage of skipping jQuery for me was to keep things fast and simple – the site was a basic blog and I felt that adding jQuery shouldn't be necessary.

That said, from a UX point of view I definitely felt an AJAX form was needed, so I set about achieving the result I wanted using plain old JavaScript, mindful that if my solution ended up unwieldy it'd be self-defeating given my reasons for not using jQuery in the first place.

###### Handling browser inconsistencies

This is the area where jQuery shines so it's an important thing to consider when you're considering going alone. This really needs to be handled on a case by case basis, but the solution here *really* is simple – we can wrap our code in a conditional statement checking support for what we need, and if there's any issue here the form will fallback to its basic POST implementation. You can view the use of AJAX as progressive enhancement or the fallback to POST as graceful degradation, it really doesn't matter here.

The thing we're going to check for is the [FormData](https://developer.mozilla.org/en/docs/Web/API/FormData) interface. It makes things very easy, but it's not supported in IE 9 or below.

While checking for FormData support, we'll also check that there's a form in the document's [forms collection](https://developer.mozilla.org/en-US/docs/Web/API/Document/forms). Note that `document.forms[0]` is only looking for the first instance of a form – you'll need to adapt this if you're likely to have more than one form on a page.

{% highlight javascript %}
if (document.forms[0] && window.FormData) { ... }
{% endhighlight %}

Really this can be split into three simple parts:

1. Some simple assignments (defining the status messages and creating a container for them)
2. Creating a new instance of [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest), specifying the type of request with its `.open` and `.setRequestHeader` methods
3. Listening for the submit event and then:
	- Preventing the default submission
	- Attaching the container we created earlier for our status messages
	- Grabbbing the data from our form with the FormData object
	– Sending the data with the XMLHttpRequest's `.send` method
	- Listening for the response with the `.onreadystatechange` method and displaying the relevant status message accordingly.

###### The full code

{% highlight javascript %}

if (document.forms[0] && window.FormData) { ... }

	var message = new Object();
	message.loading = 'Loading...';
	message.success = 'Thank you. Application received!';
	message.failure = 'Whoops! There was a problem sending your message.';

	var form = document.forms[0];

	var statusMessage = document.createElement('div');
	statusMessage.className = 'status';

	// Set up the AJAX request
	var request = new XMLHttpRequest();
	request.open('POST', '//formspree.io/hello@letsdance.agency', true);
	request.setRequestHeader('accept', 'application/json');

	// Listen for the form being submitted
	form.addEventListener('submit', function(evt) {
	    evt.preventDefault();
	    form.appendChild(statusMessage);

	    // Create a new FormData object passing in the form's key value pairs (that was easy!)
	    var formData = new FormData(form);

	    // Send the formData
	    request.send(formData);

	    // Watch for changes to request.readyState and update the statusMessage accordingly
	    request.onreadystatechange = function () {
	        // <4 =  waiting on response from server
	        if (request.readyState < 4)
	            statusMessage.innerHTML = message.loading;
	        // 4 = Response from server has been completely loaded.
	        else if (request.readyState === 4) {
	            // 200 - 299 = successful
	            if (request.status == 200 && request.status < 300)
	                statusMessage.innerHTML = message.success;
	            else
	                form.insertAdjacentHTML('beforeend', message.failure);
	        }
	    }
	});

}
{% endhighlight %}

#### The result

In the image below you can see the filled out form on the left, and the response displayed if submission is successful on the right.

![The final implementation showing an AJAX response](/images/blog/email-forms-final.jpg)



### Further reading

- [You might not need jQuery](http://youmightnotneedjquery.com) offers up lots of code examples for solving problems without jQuery