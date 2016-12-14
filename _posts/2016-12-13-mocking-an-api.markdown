---
layout:     post
title:      "A dynamic app without a server"
subtitle:   "Model, document, test and mock a new API easily"
date:       2016-12-13 00:00:00
author:     "Romina Liuzzi"
header-img: "img/api.jpg"
icon-img:   "img/icons/android.jpg"
---

<h2>Motivation</h2>
<p>As a an App Developer it isn't uncommon to need to mock an API for a new project whose API hasn't been developed yet. You could also want to use one like the marvel ones used in many meetups that allows you to create a fully functional app without getting your hands dirty on the backend.</p>
<p>In this particular case I need an out of the box solution that allows me to easily <b>model, document, test and mock a new API.</b> These are a few of the solutions I found out there.</p>

<h3>Option 1: Swagger</h3>
<p> Swagger is sometimes incorrectly categorized as a modeling language, truly it is so much more. It uses OpenAPI specification which is both machine and human readible. It counts with massive adoption and community support.</p>
<p> Swagger is supported by most free/commercial options I researched so the definition written with these tools could be used to generate the server stubs on a second phase. The power of this solution lies in that it allows top-down approach where you would use the Swagger Editor to create your Swagger definition and then use the integrated Swagger Codegen tools to generate server implementation.</p>

<h3>Option 2: Mockable.io</h3>
<p>Mockable.io seems to be a good option for mocking an API when you need to develop a client without worrying too much about the backend. The console is easy to use, allows collaboration and seems to integrate with Swagger. They have free options and trials which is really nice.</p>
<img src="/img/posts/mockable-io.png">
 <p>Another cool feature is the template assistant that allows you to automate common tasks easily, i.e: Retrieve headers from the request, return dynamic responses, etc.</p>
<img src="/img/posts/mockable-io-2.png">


<h3>Option 3: Apiary</h3>
<p>This to me looks like the best of both worlds, it supports API Blueprint and OpenAPI Modeling languages from an easy to use UI.</p> 
<p>Here you have the convenience of controling the API from the Dashboard with easy access to Documentation, mocks, tests and even an editor that commits and pushes to your github repo. But if you decide to move on to the actual API implementation you can still reuse all the work done in this stage.</p>

<p>It integrates with Github meaning your API definition is versionated out of the box.</p>
<p>It integrates with Dredd a language agnostic API testing tool. Dredd integrates with most common Continuous Integration platforms: Travis CI, CircleCI, Jenkins, ... In terms this means that your tests can be used to test the actual implementation of the API still complies with the contract once it's implemented.</p>
<p>They offer free plan and trial on premium plan so you can try the product.</p>


<h3>Benefits</h3>
<p>As an app developer for me the most immediate benefit is to be able to concentrate on developing a fully functional client with minimum effort invested on the backend. However my backend developer background makes me appreciate the collateral benefits derived from this solution:</p>
<ul>
  <li>Debate over API model is encouraged from kickoff</li>
  <li>A well modeled API is easier to maintain</li>
  <li>The API specs are written from the consumers point of view</li>
  <li>Backend developers can agree on the contract before a single line of code is implemented</li>
</ul>

<h3>Resources</h3>
[Mockable.io](https://www.mockable.io/)
[Swagger.io](http://swagger.io/)
[Apiray.io](https://apiary.io/)
[Dredd](http://dredd.readthedocs.io/en/latest/)






