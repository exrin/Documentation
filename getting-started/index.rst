Getting Started
===============

Install Nuget Package
---------------------

Search for Exrin in Nuget and install into the Native Project and the common PCL.

Initialize Framework
--------------------

In your native project, after the Xamarin Forms Init, add in the following code

.. sourcecode:: csharp

    Exrin.Framework.App.Init();

This sets up your project and you are now free to use whatever parts of Exrin you want. If you want to run your entire project on the Exrin Framework, follow all the links below.

.. toctree::

    wireup.rst
	pages.rst
	models.rst
	viewmodels.rst
	stacks.rst
	bootstrapper.rst
	launching.rst

