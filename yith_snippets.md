## Fix conflict between YITH Proteo and "Elementor Header and Footer Blocks" plugin
```
// Fixe conflict between Proteo and Custom Header and Footer BLocks
function eg_re_add_container(){
?>
<div id="content" class="site-content">
	<?php do_action( 'yith_proteo_before_page_content' ); ?>
	<div class="container">
		<?php echo yith_proteo_get_sidebar_position() ? '<div class="row">' : ''; ?>
		<?php
							  }
add_action ('get_header','eg_re_add_container',15);

function eg_re_add_container_closing(){
		?>
	</div><!-- .container -->
</div><!-- #content -->		
<?php
}
add_action ('get_footer','eg_re_add_container_closing',5);
```

