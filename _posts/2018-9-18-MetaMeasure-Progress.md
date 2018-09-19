----
layout: post
title: "MetaMeasure — Progress"
date: 2018-09-18 08:20:00 -0600
categories: learning
tags:
- C++
- MetaMeasure
---

# What is this project?

MetaMeasure is a header-only library that provides the ability to do dimensional analysis at compile time.
What do I mean when I say this? Simple: imagine if, instead of plain numbers, you could use actual units in your code, like kilometers, or seconds.
MetaMeasure implements this!

Additionally, measurements in kilometers or seconds are not interchangable.
So you can't do something like `MetaMeasure::Seconds<> time = 1.0_km;`.

However, you can divide and multiply measurements!

For example, `auto vel = 50.0_m/1.0_s;` will result in a measurement that has a value of 50, but knows it's in meters per second.
By treating units like variables in a math equation, and allowing them to be divided or multiplied, you're doing dimensional analysis.

And if you wanted to figure out how far you'd travel in a certain amount of time at that speed? `auto dist = vel*5.0_s;`

What if you want it in miles for some reason?

Easy: `MetaMeasure::Miles<> dist = vel*5.0_s;` 

# Why?

I've always been intrigued by the examples of user defined literals giving examples like `1.0_kg`, and have always wanted to see how I can implement dimensional analysis with these.

I know there are other libraries out there that do this sort of thing, but I wanted to implement one myself.
        
# Challenges

The biggest challenge in this project was the hefty amount of template metaprogramming. A lot of it was just recursive iteration over a parameter pack, or a tuple, like so:

{% highlight cpp %}
template<typename Tuple, typename Tuple2>
struct MultiplyDimensions_;

template<typename... Ts>
struct MultiplyDimensions_ <std::tuple<Ts...>, std::tuple<>>
{
  using Type = std::tuple<Ts...>;
};

template<typename... Ts, typename U, typename... Us>
struct MultiplyDimensions_<std::tuple<Ts...>, std::tuple<U, Us...>>
{
private:
  static constexpr bool TupleHasDimension = HasType
  <
    typename U::Dimension::Identifier,
    typename Ts::Dimension::Identifier...
  >::value;

public:
  using Type = typename MultiplyDimensions_
  <
    std::conditional_t
    <
      TupleHasDimension,
      MultiplyDimension<std::tuple<Ts...>, U>,
      TupleCat<std::tuple<Ts...>, std::tuple<U>>
    >,
    std::tuple<Us...>
  >::Type;
};

template<typename Tuple, typename Tuple2>
using MultiplyDimensions = RemoveZeroDimensions
<
  typename MultiplyDimensions_
  <
    LargerTuple<Tuple, Tuple2>,
    SmallerTuple<Tuple, Tuple2>
  >::Type
>;
{% endhighlight %}

The above is the metafunction I use to multiply the units of a measurement.
If a measurement is in meters per second, and I multiply it by seconds, then that should evaluate to a measurement in just meters.
If a measurement is in meters per second squared, and I multiply that by seconds, then that should evaluate to a mmeasurement in meters per second.
If a measurement is in meters, and I multiply it by millimeters, then that should evaluate to meters squared, but handle the conversion from millimeters to meters.
This metafunction, and the `operator*` overloads of `MetaMeasure::Measurement` handle this.

Actually fully implementing multiplication and division (ontop the other metafunctions they use) took hundreds of lines of the same kind of recursive stuff.
Hopefully I can find better ways to do this kind of stuff! In the mean time though, this is the best I can do.
What's challenging about doing this kind of programming is that compiler errors are often very imprecise, long, and uninformative.
In the case of MSVC, I could have a semi-colon missing somewhere, and instead of telling me about it, it'd say all of the classes/functions in my Utility.hpp file were wrong in some way when they weren't.
What a mess, honestly.

# What comes next

So here's the features right now:
* Can do dimensional analysis using the `*` and `/` operators.
* Can multiply/divide measurements by plain numbers using the `*` and `/` operators.
* Can divide a plain number by a measurement, which results in a measurement with reciprocal units.
* Can convert between units when copying, either through assignment or a constructor.
* All of the SI base units, with metric prefixes.
* Customary units of length.
* Can create your own units.
* Can create your own dimensions or types of measurement (i.e. if you want to measure network speeds, you can make a dimension for that)

What I want to add:
* All metric units, even if they're not base units, with metric prefixes
* All customary units
* More concise namespacing (instead of a namespace for just metric stuff, perhaps namespace by type of unit as well)
* Documentation, so that people can actually use this if they want to

After I've got that, I think I'll be done with this. Who knows though, maybe I'll think of more stuff to add.

# Want to check MetaMeasure out?

Cool, [click here!](https://github.com/hexagon62/MetaMeasure)
