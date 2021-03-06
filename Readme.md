# Installing

To use cache busting you need to do the following

nuget : ```Install-Package Gibe.CacheBusting```

If you're using **[Autofac](https://autofac.org/"Autofac")**, you'll need the following

nuget : ```Gibe.CacheBusting.Autofac```

Bindings registration: ```builder.RegisterModule<Gibe.CacheBusting.Autofac.DefaultBindingsModule>();```

If you're using **[LightInject](https://www.lightinject.net/"LightInject")**, you'll need the following

nuget : ```Gibe.CacheBusting.LightInject```

Bindings registration: ```builderGibe.CacheBusting.LightInject.DefaultBindingsModule.RegisterServicesFactories(container);```

If you're using **[Ninject](http://www.ninject.org/"Ninject")**, you'll need the following

nuget : ```Gibe.CacheBusting.Ninject```

Bindings registration: ```kernel.Load<Gibe.CacheBusting.Ninject.DefaultBindingsModule>();```

The bindings for any other inversion of control container are:

```Gibe.CacheBusting.IRevisionManifest``` to ```Gibe.CacheBusting.RevisionManifest```
```Gibe.CacheBusting.IManifestFileFactory``` to ```Gibe.CacheBusting.ConfigManifestFileFactory```
```Gibe.FileSystem.IDirectoryService``` to ```GibeGibe.FileSystem.DirectoryService```
```Gibe.FileSystem.IFileService``` to ```GibeGibe.FileSystem.FileService```
```Gibe.FileSystem.IPaths``` to ```GibeGibe.FileSystem.Paths```

You will need to add [gulp-rev](https://www.npmjs.com/package/gulp-rev"gulp-rev"), [gulp-rev-delete-original](https://www.npmjs.com/package/gulp-rev-delete-original"gulp-rev-delete-original") and [gulp-filter](https://www.npmjs.com/package/gulp-filter"gulp-filter") to your package.json

Update the gulp file to then include rev(), revdel() and rev.manifest() in the process for building css and js

For example:

```javascript
var gulp = require('gulp'),
	bower = require('gulp-bower'),
	sass = require('gulp-sass'),
	sourcemaps = require('gulp-sourcemaps'),
	concat = require('gulp-concat'),
	uglify = require('gulp-uglify'),
	rev = require('gulp-rev'),
	revdel = require('gulp-rev-delete-original'),
	filter = require('gulp-filter');

gulp.task('css', function () {
	return gulp.src('./gulp/scss/**/*.scss')
		.pipe(sourcemaps.init())
		.pipe(sass({ outputStyle: 'compressed' }))
		.pipe(sourcemaps.write('.'))
		.pipe(gulp.dest('./css'))
		.pipe(filter('**/*.css'))
		.pipe(rev())
		.pipe(revdel())
		.pipe(gulp.dest('./css'))
		.pipe(rev.manifest())
		.pipe(gulp.dest('./css'));
});

gulp.task('js', function () {
	return gulp.src([
			'./gulp/scripts/scripts-lib/**/*.js',
			'./gulp/scripts/history.js/history.js',
			'./gulp/scripts/history.js/history.adapter.jquery.js',
			'./gulp/scripts/plugins/**/*.js',
			'./gulp/scripts/site.js',
			'./gulp/scripts/modules/**/*.js'
		])
		.pipe(sourcemaps.init())
		.pipe(concat('site.js'))
		.pipe(uglify())
		.pipe(sourcemaps.write('.'))
		.pipe(gulp.dest('./js'))
		.pipe(filter('**/*.js'))
		.pipe(rev())
		.pipe(revdel())
		.pipe(gulp.dest('./js'))
		.pipe(rev.manifest())
		.pipe(gulp.dest('./js'));
});

gulp.task('build', ['css', 'js']);

gulp.task('watch', function () {
	gulp.watch('./gulp/scss/**/*.scss', ['css']);
	gulp.watch('./gulp/scripts/**/*.js', ['js']);
});
```

Then you can add to web.config <configSections>
```xml
<section name="cacheBusting" type="Gibe.CacheBusting.Config.CacheBustingSection, Gibe.CacheBusting" />
```
And add this section somewhere in <configuration> in your web.config

```xml
<cacheBusting>
	<manifests>
		<add path="/css/" file="~/css/rev-manifest.json" />
		<add path="/js/" file="~/js/rev-manifest.json" />
	</manifests>
</cacheBusting>
```

Add to your ~/views/web.config : ```xml <add namespace="Gibe.CacheBusting" />```

Use in your views ```@Url.Asset("/js/site.js")```

This will automatically change the URL to the hashed version if found in the manifest.