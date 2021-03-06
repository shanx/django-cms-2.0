Plugins
=======

Lets say we have the following gallery model:

	class Gallery(models.Model):
		name = models.CharField(max_length=30)
	
	class Picture(models.Model):
		gallery = models.ForeignKey(Gallery)
		image = models.ImageField(upload_to="uploads/images/")
		description = models.CharField(max_lenght=60)

And we want to display this gallery between two text blocks.
We can do this with a cms plugin.
To create a cms plugin you need two components: A CMSPlugin model and a cms_plugins.py

Plugin Model
------------

First create a model that links the gallery:

	from cms.models import CMSPlugin

	class GalleryPlugin(CMSPlugin):
		gallery = models.ForeignKey(Gallery)
	

Be sure to extend from CMSPlugin instead of models.Model.
The plugin model can have any fields it wants. They are the fields that
get displayed if you edit the plugin.

cms_plugins.py
--------------

After that create in the application folder (the same the models.py is) a cms_plugins.py

In there write the following:

	from cms.plugin_base import CMSPluginBase
	from cms.plugin_pool import plugin_pool
	from models import GalleryPlugin
	from django.utils.translation import ugettext as _

	class CMSGalleryPlugin(CMSPluginBase):
		model = GalleryPlugin
    	name = _("Gallery")
    	render_template = "gallery/gallery.html"
    
    	def render(self, context, instance, placeholder):
        	return context.update({'gallery':instance.gallery, 'placeholder':placeholder})
    
	plugin_pool.register_plugin(CMSGalleryPlugin)		

### model ###

is the CMSPlugin Model we created earlier
If you don't need a model because you just want to display some template logic use CMSPlugin from cms.models as the model instead.

### name ###

will be displayed in the plugin editor

### render_template ###

will be rendered with the context returned by the render function

### render ###

the render function takes 3 arguments:

**context**:

the context of the template placeholder was placed.

**instance**:

the instance of the GalleryPlugin model

**placeholder**:

the name of the placeholder this plugin appears.
It is normally a good idea to give the placeholder to the template so you can style
the content differently in the template based on which placholder it is placed.

If you want to process forms in the render function or if you wan't to see if the user is logged in you may want to access the request. 
You can accomplish this simply with:

	request = context['request']

because the request will always be in the context as the requescontext processor is required by the cms

Template
--------
now create a gallery.html template in templates/gallery/ and write the following in there.

	{% for image in gallery.picture_set.all %}
		<img src="{{ image.image.url }}" alt="{{ image.description }}" />
	{% endfor %}

Now go into the admin create a gallery and afterwards go into a page and add a gallery plugin and some pictures should appear in your page.

Advanced
--------

CMSGalleryPlugin can be even further customized:

you can add some other attributes:

### form_template ###

the template used to render the form.
It is good practice to extend from "admin/cms/page/plugin_forms.html" So you get the same look and feel.

### form ###

A reference to a Form that will be displayed if you edit the plugin. If nothing is provided a model form of the model
will be used.

### placeholders ###

a tuple of placeholder names this plugin can appear:

For exmaple you have 3 placeholders: content, right-column, left-column
And you want that your plugin only appears in either the left or right column but no in content:

add this to your plugin: 
	
	placeholders = ('right-column', 'left-column')
