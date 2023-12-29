<!doctype html>

<head>
	<title></title>
	<meta name="viewport" content="width = 1050, user-scalable = no" />
	<script type="text/javascript" src="../../extras/jquery.min.1.7.js"></script>
	<script type="text/javascript" src="../../extras/modernizr.2.5.3.min.js"></script>
	<script type="text/javascript" src="../../lib/hash.js"></script>

</head>

<body>
	<div id="canvas">
		<div class="magazine-viewport">
			<div class="container">
				<div class="magazine">
					<div ignore="1" class="next-button"></div>
					<div ignore="1" class="previous-button"></div>
				</div>
			</div>
		</div>
		<div class="thumbnails">
			<div>
				<ul>
					<li class="i">
						<img src="pages/1-thumb.jpg" width="76" height="100" class="page-1">
						<span>1</span>
					</li>
					<li class="d">
						<img src="pages/2-thumb.jpg" width="76" height="100" class="page-2">
						<img src="pages/3-thumb.jpg" width="76" height="100" class="page-3">
						<span>2-3</span>
					</li>
					<li class="d">
						<img src="pages/4-thumb.jpg" width="76" height="100" class="page-4">
						<img src="pages/5-thumb.jpg" width="76" height="100" class="page-5">
						<span>4-5</span>
					</li>
					<li class="d">
						<img src="pages/6-thumb.jpg" width="76" height="100" class="page-6">
						<img src="pages/7-thumb.jpg" width="76" height="100" class="page-7">
						<span>6-7</span>
					</li>
					<li class="d">
						<img src="pages/8-thumb.jpg" width="76" height="100" class="page-8">
						<img src="pages/9-thumb.jpg" width="76" height="100" class="page-9">
						<span>8-9</span>
					</li>
					<li class="d">
						<img src="pages/10-thumb.jpg" width="76" height="100" class="page-10">
						<img src="pages/11-thumb.jpg" width="76" height="100" class="page-11">
						<span>10-11</span>
					</li>
					<li class="d">
						<img src="pages/10-thumb.jpg" width="76" height="100" class="page-10">
						<img src="pages/11-thumb.jpg" width="76" height="100" class="page-11">
						<span>12-13</span>
					</li>
					<li class="i">
						<img src="pages/12-thumb.jpg" width="76" height="100" class="page-12">
						<span>14</span>
					</li>
				</ul>
			</div>
		</div>
	</div>
</body>
<script type="text/javascript">
	function loadApp() {
		$('#canvas').fadeIn(1000);
		var flipbook = $('.magazine');
		if (flipbook.width() == 0 || flipbook.height() == 0) {
			setTimeout(loadApp, 10);
			return;
		}
		flipbook.turn({
			width: 922,
			height: 600,
			duration: 1000,
			acceleration: !isChrome(),
			gradients: true,
			autoCenter: true,
			elevation: 50,
			pages: 12,
			when: {
				turning: function (event, page, view) {
					var book = $(this),
						currentPage = book.turn('page'),
						pages = book.turn('pages');
					Hash.go('page/' + page).update();
					disableControls(page);
					$('.thumbnails .page-' + currentPage).
						parent().
						removeClass('current');
					$('.thumbnails .page-' + page).
						parent().
						addClass('current');
				},
				turned: function (event, page, view) {
					disableControls(page);
					$(this).turn('center');
					if (page == 1) {
						$(this).turn('peel', 'br');
					}
				},
				missing: function (event, pages) {
					for (var i = 0; i < pages.length; i++)
						addPage(pages[i], $(this));
				}
			}
		});
		Hash.on('^page\/([0-9]*)$', {
			yep: function (path, parts) {
				var page = parts[1];
				if (page !== undefined) {
					if ($('.magazine').turn('is'))
						$('.magazine').turn('page', page);
				}
			},
			nop: function (path) {
				if ($('.magazine').turn('is'))
					$('.magazine').turn('page', 1);
			}
		});
		$(window).resize(function () {
			resizeViewport();
		}).bind('orientationchange', function () {
			resizeViewport();
		});
		$('.thumbnails').click(function (event) {
			var page;
			if (event.target && (page = /page-([0-9]+)/.exec($(event.target).attr('class')))) {

				$('.magazine').turn('page', page[1]);
			}
		});
		$('.thumbnails li').
			bind($.mouseEvents.over, function () {
				$(this).addClass('thumb-hover');
			}).bind($.mouseEvents.out, function () {
				$(this).removeClass('thumb-hover');
			});
		if ($.isTouch) {
			$('.thumbnails').
				addClass('thumbanils-touch').
				bind($.mouseEvents.move, function (event) {
					event.preventDefault();
				});
		} else {
			$('.thumbnails ul').mouseover(function () {
				$('.thumbnails').addClass('thumbnails-hover');
			}).mousedown(function () {
				return false;
			}).mouseout(function () {
				$('.thumbnails').removeClass('thumbnails-hover');
			});
		}
		if ($.isTouch) {
			$('.magazine').bind('touchstart', regionClick);
		} else {
			$('.magazine').click(regionClick);
		}
		$('.next-button').bind($.mouseEvents.over, function () {
			$(this).addClass('next-button-hover');
		}).bind($.mouseEvents.out, function () {
			$(this).removeClass('next-button-hover');
		}).bind($.mouseEvents.down, function () {
			$(this).addClass('next-button-down');
		}).bind($.mouseEvents.up, function () {
			$(this).removeClass('next-button-down');
		}).click(function () {
			$('.magazine').turn('next');
		});
		$('.previous-button').bind($.mouseEvents.over, function () {
			$(this).addClass('previous-button-hover');
		}).bind($.mouseEvents.out, function () {
			$(this).removeClass('previous-button-hover');
		}).bind($.mouseEvents.down, function () {
			$(this).addClass('previous-button-down');
		}).bind($.mouseEvents.up, function () {
			$(this).removeClass('previous-button-down');
		}).click(function () {
			$('.magazine').turn('previous');
		});
		resizeViewport();
		$('.magazine').addClass('animated');
	}
	yepnope({
		test: Modernizr.csstransforms,
		yep: ['../../lib/turn.js'],
		nope: ['../../lib/turn.html4.min.js'],
		both: ['../../lib/zoom.min.js', 'js/magazine.js', 'css/magazine.css'],
		complete: loadApp
	});
	// ture banner slideshow magazine magezoom previous  addclass butdown
	// loadapp complete extras 
	$(window).resize(function () {
			resizeViewport();
		}).bind('orientationchange', function () {
			resizeViewport();
		});
		$('.thumbnails').click(function (event) {
			var page;
			if (event.target && (page = /page-([0-9]+)/.exec($(event.target).attr('class')))) {

				$('.magazine').turn('page', page[1]);
			}
		});

		/*
			function dara(){
				let magaine = window.on scrollTop || windwos.documentBodyHeight
			  	let mapData = data.map((item,index)=>{
					return	item.id = index+1
				})
				$('.thumbnails').removeClass('thumbnails-hover');
			}
			console.log(data._photo_.type)
			
			

		*/

</script>

</html>
