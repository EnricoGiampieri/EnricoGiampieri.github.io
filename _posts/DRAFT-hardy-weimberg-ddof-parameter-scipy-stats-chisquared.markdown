---
layout: post
title:  "hardy Weinberg equilibrium (or the ddof parameter in scipy.stats.chisquared)"
date:   DRAFT
categories: introduction
tags: [python]
--- 

Just a post to clarify an issue I encountered today in some explorative analisys of a medical database that made me wonder why the documentation of many functions doesn't include examples...

I have the allelic type of a population for some gene. The one I am interested into has three different alleles, a1, a2 and a3, each person can be of one of the following categories:

* a1/a1
* a1/a2
* a1/a3
* a2/a2
* a2/a3
* a3/a3

as order is not meaningful. These data are in pandas DataFrame in which each row is a patient (3000, more or less).

What I want to do is estimate if these patients follows the [Hardy-Wienberg equilibrium](http://en.wikipedia.org/wiki/Hardy%E2%80%93Weinberg_principle), that is the null hypothesis about the gene distribution given the allele frequency. If the allele is almost neutral we have a precise expected distribution of the couples under the hypothesis of random mating.

The first step is to evaluate these frequencies, then use them to evaluate the expected observation, confronting them with a chisquared test.

The gene is APOE, so this is the name of the DataFrame column. what I can do to get the expected frequencies is this:

{% highlight python %}
s0 = list(data['APOE'].dropna().apply(lambda s: s.split('/')[0]).values)
s1 = list(data['APOE'].dropna().apply(lambda s: s.split('/')[1]).values)
observed_alleli = Series(Counter( s0+s1 ))
{% endhighlight %}

The first two lines create a list from the first and second element of the allele couple, then the third feed it to a Counter and convert it to a pandas Serie.

The result is this:

* e1 452 
* e2 4932 
* e3 630

Now to obtain the expected frequencies we divide these observation for the total number of patients, than use these frequencies to create the expected observation of each class using the combination of multiple alleles that can be found on wikipedia

{% highlight python %}
pe1 = frequenze_alleli['e1']
pe2 = frequenze_alleli['e2']
pe3 = frequenze_alleli['e3']
teorical = Series({'e1/e1': pe1**2, 'e1/e2': 2*pe1*pe2, 'e1/e3': 2*pe1*pe3,
                   'e2/e2': pe2**2, 'e2/e3': 2*pe2*pe3, 'e3/e3': pe3**2})
{% endhighlight %}

Now, I put them togheter on a dataframe and test the hypothesis of the equilibrium with the function scipy.stats.chisquared

{% highlight python %}
observed_totali = data.groupby('APOE')['APOE'].count()
teorical*=sum(observed_totali)
teorical = teorical.apply(round)
APOE_table = DataFrame({'obs':observed_totali,'exp':teorical}).dropna()
{% endhighlight %}

The pvalue is the second value returned by the function:

{% highlight python %}
st.chisquare(APOE_table.obs.values, APOE_table.exp.values)[1]
{% endhighlight %}

But wait! we estimated 3 paremeters from our initial distribution, so we have to reduce by 3 our degree of freedom or we are going to have a biased result! Now, the function has a parameter ddof that is documented as "adjustment to the degrees of freedom for the p-value". Now, it's very easy to overlook the exact wording and get confused: this is not the actual number of degree of freedom that we use, but rather the number of degree of freedom that we remove due to the fitting procedure.

This mean that if you have k categories (6 in this case) and 3 estimeted parametes you don't need to insert k-1-3 (the actual number of degree of freedom of the test) but rather simply 3 (the parameters that you estimated).

This is very useful as I don't need to count the number of degree of freedom by hand but rather just insert the number of parameter that I expect. After a couple of try and error and a stackoverflow research I found out the correct interpretation and felt rather stupid, but then I asked myself: It is so difficult to put some example in this function? is a commonly used function, and one example speak more loudly than a thousand words of theoretical explanation.

In the next days I will try to commit something, but the lesson here is: documentation is boring, good documentation is hard, clear documentation is a royal pain, but this is not an excuse. Better write a good documented but simple function that a powerful but undocumented one. Your users are going to learn your library trough the examples you give them, so if the example are too few or are plain wrong they are not going to learn to use it well. Even worst, if you feel that writing an example would be too boring because you need a lot of boilerplate code, that is a very strong indication that your API is badly designed!

Be advised that if a user can't get a good feeling with your library in the first minute  (i.e. the introduction of the library and the examples in the functions documentation) I will not try to read more carefully your source code, but rather search for other library easier to use!

And remember that a badly documented feature is a bug, as it introduce unexpected error given some correct (from the user point of view) data.