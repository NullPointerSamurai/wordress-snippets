## Fix conflict between YITH Proteo and "Elementor Header and Footer Blocks" plugin
```
// Fix conflict between Proteo and Custom Header and Footer BLocks
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

## Add YITH Wishlist counter icon via Shortcode
```
//Add YITH Wishlist counter icon via Shortcode
if ( defined( 'YITH_WCWL' ) && ! function_exists( 'yith_wcwl_get_items_count' ) ) {
    function yith_wcwl_get_items_count() {
      ob_start();
      ?>
        <a href="<?php echo esc_url( YITH_WCWL()->get_wishlist_url() ); ?>">
          <span class="yith-wcwl-items-count">
            <i class="yith-wcwl-icon fa fa-heart-o"><?php echo esc_html( yith_wcwl_count_all_products() ); ?></i>
          </span>
        </a>
      <?php
      return ob_get_clean();
    }
  
    add_shortcode( 'yith_wcwl_items_count', 'yith_wcwl_get_items_count' );
  }
  
  if ( defined( 'YITH_WCWL' ) && ! function_exists( 'yith_wcwl_ajax_update_count' ) ) {
    function yith_wcwl_ajax_update_count() {
      wp_send_json( array(
        'count' => yith_wcwl_count_all_products()
      ) );
    }
  
    add_action( 'wp_ajax_yith_wcwl_update_wishlist_count', 'yith_wcwl_ajax_update_count' );
    add_action( 'wp_ajax_nopriv_yith_wcwl_update_wishlist_count', 'yith_wcwl_ajax_update_count' );
  }
  
  if ( defined( 'YITH_WCWL' ) && ! function_exists( 'yith_wcwl_enqueue_custom_script' ) ) {
    function yith_wcwl_enqueue_custom_script() {
      wp_add_inline_script(
        'jquery-yith-wcwl',
        "
          jQuery( function( $ ) {
            $( document ).on( 'added_to_wishlist removed_from_wishlist', function() {
              $.get( yith_wcwl_l10n.ajax_url, {
                action: 'yith_wcwl_update_wishlist_count'
              }, function( data ) {
                $('.yith-wcwl-items-count').children('i').html( data.count );
              } );
            } );
          } );
        "
      );
    }
  
    add_action( 'wp_enqueue_scripts', 'yith_wcwl_enqueue_custom_script', 20 );
  }
```
