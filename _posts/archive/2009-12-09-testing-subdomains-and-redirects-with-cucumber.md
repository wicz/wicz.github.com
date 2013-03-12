---
layout: post
title: Testing subdomains and redirects with Cucumber
published: false
---

__UPDATE 2011-04-13:__ If you're on Mac OS X, forget all the /etc/hosts and PAC thing and go for [Pow](http://pow.cx/).

The other day I was writing some Cucumber features for a Rails app that relies on subdomains for creating per-user contexts. It seems that Webrat can't handle subdomains and redirects the way I expected.

A Little Background
-------------------

I hate when I have to access 127.0.0.1 or localhost:3000. Typing all these numbers, dots and colon doesn't feel comfortable at all. I like domain names, and since I am developing a subdomain-aware app, I want to be able to access using names. That said, I configured my Firefox with a PAC file that routes *.local to my server running at 127.0.0.1:3000 (oh man, if you're still using /etc/hosts you're so 2000-and-late! =p).

Here's how it looks like. You can grab more details here.

{% highlight javascript linenos=table linenospecial=1 %}
function FindProxyForURL(url, host) {
  if (shExpMatch(host, "*.local")) {
    return "PROXY 127.0.0.1:3000";
  }
  return "DIRECT";
}
{% endhighlight %}


Now I am able to access johndoe.app.local, foobar.app.local, etc. No numbers, no colon, just NAMES! =)

My ApplicationController implements a simple before_filter that checks for a valid account, otherwise redirects to a fallback page. Something like this:

{% highlight ruby linenos=table linenospecial=1 %}
before_filter :account_required

def current_account
  @current_account ||= Account.find_by_subdomain(request.subdomains.first)
end

def account_required
  unless current_account
    flash[:error] = :account_not_found
    redirect_to fallback_url
  end
end
{% endhighlight %}


You can have more on subdomains as accounts here, here and here.

The Scenario
------------

The feature was a simple sign in.

{% highlight gherkin %}
Given the foobar account
When I go to foobar's sign in page
And I fill in "user_session[email]" with "foobar@test.com"
And I fill in "user_session[password]" with "1234"
And I press "Sign in"
Then I should be on dashboard
And I should see "Welcome, foobar"
{% endhighlight %}


The Caveats
-----------

By default, Webrat uses www.example.com as host for testing. Since it was trying to sign in to a account that doesn't exist (the Given creates foobar), it was being redirect to the fallback url, so it couldn't find the form fields. Knowing that Webrat's RailsAdapter uses ActionController::Integration::Session, we can easily solve this by overwriting the host name to use in the next request. Our Given should look like this:

{% highlight ruby linenos=table linenospecial=1 %}
Given /^the foobar account$/ do
  Foobar.create(...)
  host! "foobar.app.local"
end
{% endhighlight %}


Now being able to fill in the form, after submitting it, I still wasn't sent to the expected page. I was willing to see the dashboard, but it was rendering /session, which means I wasn't authenticating at all, even passing valid credentials... WTF!? Then dumping the ActionController::Response object, I verified that it was actually redirecting (@status => "302 Found"), so I decided to investigate Webrat.

Webrat's behavior is to follow any redirects, except for external ones. And subdomains aren't considered as externals. Digging a little deeper, people have reported similar problems and patches (although I've tried none) here, here, here and here. Since the problem seems that Webrat shows some really deep love to example.com, I came up with a ugly-and-fast-but-it-works solution:

Set a domain in my environment.rb:

{% highlight ruby linenos=table linenospecial=1 %}
APP_DOMAIN = "app." + case RAILS_ENV
when "production" then "com"
when "cucumber" then "example.com"
else "local"
end
{% endhighlight %}

And changed the above Given step to  `host! "foobar.#{APP_DOMAIN}"`.

Bar green, code clean, life moves on.
