---
layout: post
title: Testing dates in JS using Jasmine
description: "Testing dates using Jasmine"
tags: [unit test, jasmine, mocking date, momentjs]
image:
  feature: abstract-12.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

Testing dates in unit tests is not always an easy stuff, In this post, I'm going to show an easy and elegant way to do that in JS using Jasmine.

First, take a look at the function below. It's calculating the difference between today and a date passed by param. To make the example easier, I'm using momentJS library to work with dates.


{% highlight js %}
function getDaysBetweenTodayAndADate(date) {
  let today = moment();
  return date.diff(today); 
}
{% endhighlight %}

How can we test it? The moment object will always return the current date, how can we write a test that can pass every day?

In order to solve this problem, we can use Jasmine Clock to mock the current date.


{% highlight js %}
describe('testing dates in js', function () {

beforeEach(() => {
  let today = moment('2016-01-01').toDate();
  jasmine.clock().mockDate(today);
});

it('should return the difference between today', () => {
  let date = moment('2016-01-05');
  expect(getDaysBetweenTodayAndADate(date), 4);
});

});
{% endhighlight %}


If you are using Jasmine as your test tool, it’s an easy way to mock dates!

However, if you are not using Jasmine, there is another way to solve this problem, you can create a date utility class and mock it. I usually do it using Java for example, but it’s a topic for another day.

I hope you enjoyed this post!

Until next time!

