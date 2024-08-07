> [!IMPORTANT]
> Compatible with WooCommerce version 8.0 and higher.
## Visual Hooks
https://www.businessbloomer.com/woocommerce-visual-hook-guide-single-product-page/
https://www.businessbloomer.com/woocommerce-visual-hook-guide-archiveshopcat-page/

## General
## Add maintenance mode in WordPress but exclude administrator v2

```php
// Add maintenance mode in WordPress but exclude administrator
add_action( 'template_redirect', 'maintenance_mode_redirect' );
function maintenance_mode_redirect() {
    if ( ! is_admin() && ! is_admin_user_logged_in() && ! is_user_logged_in() ) {
        wp_redirect( home_url( '/maintenance.php' ) ); // you need to create an maintenance.php page first
        exit();
    }
}

function is_admin_user_logged_in() {
    $user = wp_get_current_user();
    return $user && array_intersect( $user->roles, array( 'administrator' ) );
}
```

### delete_all_wc_products_and_images
```php

<?php
require_once('wp-load.php'); // Adjust the path to find the correct wp-load.php if necessary

function eg_delete_all_wc_products_and_images() {
    // Get all WooCommerce products
    $args = array(
        'post_type'      => 'product',
        'posts_per_page' => -1, // Retrieve all products
        'fields'         => 'ids' // Only get product IDs to save memory
    );

    $products = get_posts($args);

    foreach ($products as $product_id) {
        // Get all attached image IDs
        $image_ids = array();
        $image_ids[] = get_post_thumbnail_id($product_id); // Featured image
        $gallery_images = get_post_meta($product_id, '_product_image_gallery', true);

        if (!empty($gallery_images)) {
            $image_ids = array_merge($image_ids, explode(',', $gallery_images));
        }

        // Delete all images associated with the product
        foreach ($image_ids as $image_id) {
            if (!empty($image_id)) {
                wp_delete_attachment($image_id, true);
            }
        }

        // Finally, delete the product
        wp_delete_post($product_id, true);
    }
}

// Run only once and then DISABLE again
// eg_delete_all_wc_products_and_images();

?>
```

### Automatically delete WC images (main+gallery) from Server after deleting a product

```php
// Automatically delete WC images from Server after deleting a product
add_action( 'before_delete_post', 'eg_delete_product_images', 10, 1 );
function eg_delete_product_images( $post_id )
{
    $product = wc_get_product( $post_id );

    if ( !$product ) {
        return;
    }

    $featured_image_id = $product->get_image_id();
    $image_galleries_id = $product->get_gallery_image_ids();

    if( !empty( $featured_image_id ) ) {
        wp_delete_post( $featured_image_id );
    }

    if( !empty( $image_galleries_id ) ) {
        foreach( $image_galleries_id as $single_image_id ) {
            wp_delete_post( $single_image_id );
        }
    }
}

```
### Send e-mail to client for cancelled order WC
```php
/** Send e-mail to client for cancelled order **/ 
add_action('woocommerce_order_status_changed', 'eg_send_custom_email_notifications', 10, 4);
function eg_send_custom_email_notifications($order_id, $old_status, $new_status, $order)
{
    if ($new_status == 'cancelled' || $new_status == 'failed') {
        $wc_emails = WC()->mailer()->get_emails(); // Get all WC_emails objects instances
        $customer_email = $order->get_billing_email(); // The customer email
    }

    if ($new_status == 'cancelled') {
        // change the recipient of this instance
        $wc_emails['WC_Email_Cancelled_Order']->recipient = $customer_email;
        // Sending the email from this instance
        $wc_emails['WC_Email_Cancelled_Order']->trigger($order_id);
    } elseif ($new_status == 'failed') {
        // change the recipient of this instance
        $wc_emails['WC_Email_Failed_Order']->recipient = $customer_email;
        // Sending the email from this instance
        $wc_emails['WC_Email_Failed_Order']->trigger($order_id);
    }
}
```
> [!NOTE]
> This code uses the WooCommerce woocommerce_order_status_changed hook to call the send_cancelled_order_email function when an order is canceled.

> The send_cancelled_email_notifications function then checks if the new status is either “cancelled" or “failed". If the new status matches either of these values, the function gets all instances of the WC_Emails class (which are responsible for sending WooCommerce emails) and the email address of the customer who placed the order.

> If the new status is “cancelled", the recipient of the WC_Email_Cancelled_Order instance (which is responsible for sending the “Order Cancelled" email) is set to the customer email, and the email is triggered with the trigger function. If the new status is “failed", the recipient of the WC_Email_Failed_Order instance (which is responsible for sending the “Order Failed" email) is set to the customer email, and the email is triggered with the trigger function.

### Display the discount percentage on the sale badge WC

```php
/** Display the discount percentage on the sale badge WC**/
add_filter('woocommerce_sale_flash', 'add_percentage_to_sale_badge', 20, 3);
function add_percentage_to_sale_badge($html, $post, $product)
{

    if ($product->is_type('variable')) {
        $percentages = array();

        // Get all variation prices
        $prices = $product->get_variation_prices();

        // Loop through variation prices
        foreach ($prices['price'] as $key => $price) {
            // Only on sale variations
            if ($prices['regular_price'][$key] !== $price) {
                // Calculate and set in the array the percentage for each variation on sale
                $percentages[] = round(100 - (floatval($prices['sale_price'][$key]) / floatval($prices['regular_price'][$key]) * 100));
            }
        }
        // We keep the highest value
        $percentage = max($percentages) . '%';
    } elseif ($product->is_type('grouped')) {
        $percentages = array();

        // Get all variation prices
        $children_ids = $product->get_children();

        // Loop through variation prices
        foreach ($children_ids as $child_id) {
            $child_product = wc_get_product($child_id);

            $regular_price = (float) $child_product->get_regular_price();
            $sale_price    = (float) $child_product->get_sale_price();

            if ($sale_price != 0 || !empty($sale_price)) {
                // Calculate and set in the array the percentage for each child on sale
                $percentages[] = round(100 - ($sale_price / $regular_price * 100));
            }
        }
        // We keep the highest value
        $percentage = max($percentages) . '%';
    } else {
        $regular_price = (float) $product->get_regular_price();
        $sale_price    = (float) $product->get_sale_price();

        if ($sale_price != 0 || !empty($sale_price)) {
            $percentage    = round(100 - ($sale_price / $regular_price * 100)) . '%';
        } else {
            return $html;
        }
    }
    return '<span class="onsale">-' . $percentage . '</span>';
}
```
### Change WC thumbnail for sub-categories with image of the first product
```php
/** Change WC thumbnail for sub-categories with image of the first product **/
add_action('woocommerce_before_subcategory_title', 'eg_replace_category_thumbnail', 10);
function eg_replace_category_thumbnail($category)
{
    // Checking if the category has products
    $args = array(
        'post_type' => 'product',
        'posts_per_page' => 1,
        'product_cat' => $category->slug,
        'orderby' => 'date',
        'order' => 'DESC'
    );

	// store the products
    $products = new WP_Query($args);

    if ($products->have_posts()) {
        while ($products->have_posts()) : $products->the_post();
            // Get the ID of the first product
            $first_product_id = get_the_ID();

            // Get the image of the first product
            $image = wp_get_attachment_image_src(get_post_thumbnail_id($first_product_id), 'single-post-thumbnail');

            // Replace category thumbnail with first product image
            if ($image) {
                echo '<img src="' . esc_url($image[0]) . '" alt="' . esc_attr($category->name) . '" />';
            }

        endwhile;

        wp_reset_postdata();
    }
}
```
## Payment & Shipping Snippets
### Hide shipping rates when free shipping is available, but keep "Local pickup"
```php
// Hide shipping rates when free shipping is available, but keep "Local pickup"
add_filter( 'woocommerce_package_rates', 'eg_hide_shipping_when_free_is_available', 10, 2 );
function eg_hide_shipping_when_free_is_available( $rates, $package ) {
	$new_rates = array();
	foreach ( $rates as $rate_id => $rate ) {
		// Only modify rates if free_shipping is present.
		if ( 'free_shipping' === $rate->method_id ) {
			$new_rates[ $rate_id ] = $rate;
			break;
		}
	}

	if ( ! empty( $new_rates ) ) {
		//Save local pickup if it's present.
		foreach ( $rates as $rate_id => $rate ) {
			if ('local_pickup' === $rate->method_id ) {
				$new_rates[ $rate_id ] = $rate;
				break;
			}
		}
		return $new_rates;
	}

	return $rates;
}
```
### Hide specific payment method based on total weight in WooCommerce
```php
//Hide specific payment method based on total weight in Woocommerce
add_filter( 'woocommerce_available_payment_gateways', 'eg_hide_payment_gateways_based_on_weight', 11, 1 );
function eg_hide_payment_gateways_based_on_weight( $available_gateways ) {
    if ( is_admin() ) return $available_gateways; // Only on frontend

    $total_weight = WC()->cart->get_cart_contents_weight();

    if( $total_weight >= 15 && isset($available_gateways['cod']) )
        unset($available_gateways['cod']); // unset 'cod'

    return $available_gateways;
}
```

## WP All Import - Export Snippets
### Custom Fields
```php
_metaseo_metatitle
_metaseo_metadesc
_metaseo_metakeywords
_metaseo_metaspecific_keywords

we_skroutzxml_custom_availability

```

## Web Expert - Skroutz
### Display custom skroutz availability combine with webExpert plugin @ WC Single Product Page
```php
// Display custom skroutz availability combine with webExpert plugin @ WC Single Product Page
add_action("woocommerce_after_add_to_cart_button", "egr_show_avail_sktrz", 10);
function egr_show_avail_sktrz(){

    global $product;
    $product->get_id();

    // Απλό προιόν
    if ($product->is_type('simple')) {

        /**
         ** 
         ** 
         ** --- Αν το προιόν ΑΠΛΟ και είναι INSTOCK ---
         ** 
         **
         **/
        if ($product->get_stock_quantity() > 0) {

            // Αν στο προιον έχει επιλεγεί το πεδίο "Διαθεσιμότητα"
            if ($product->get_meta("we_skroutzxml_custom_availability")) {
                echo "<div class='eg_skrtz_availability'>" . $product->get_meta("we_skroutzxml_custom_availability") . "</div>";
            }

            // Αν στο προιον ΔΕΝ έχει επιλεγεί το πεδίο "Διαθεσιμότητα", είναι δηλαδή Διαθεσιμότητα = Προκαθορισμένο
            else {
                //echo "<div class='eg_skrtz_availability'>Άμεση Παραλαβή / Παράδοση 1 έως 3 ημέρες</div>";
                echo "<div class='eg_skrtz_availability eg_skrtz_in_stock'>" . get_option('we_skroutz_xml_availability') . "</div>";
            }
        } // end-if "InStock"

        /**
         ** 
         ** --- Αν το προιόν είναι OUTOFSTOCK ΚΑΙ επιτρέπει BACKORDERS ---
         ** 
         **
         **/
        if ($product->get_stock_quantity() <= 0 && $product->backorders_allowed()) {

            // Αν στο προιον έχει επιλεγεί το πεδίο "Διαθεσιμότητα"
            if ($product->get_meta("we_skroutzxml_custom_preavailability")) {
                echo "<div class='pr eg_skrtz_availability'>" . $product->get_meta("we_skroutzxml_custom_preavailability") . "</div>";
            }

            // Αν στο προιον ΔΕΝ έχει επιλεγεί το πεδίο "Διαθεσιμότητα", είναι δηλαδή Διαθεσιμότητα = Προκαθορισμένο
            else {
                echo "<div class='pree eg_skrtz_availability eg_skrtz_yes_bo'>" . get_option('we_skroutz_xml_preavailability') . "</div>";
            }
        } // end-if "OutOfStock"
    }
}
```

## Administrator Dashboard

### Add maintenance mode in WordPress but exclude administrator
```php
// Add maintenance mode in WordPress but exclude administrator
add_action( 'template_redirect', 'maintenance_mode_redirect' );
function maintenance_mode_redirect() {
    if ( ! is_admin() && ! is_admin_user_logged_in() && ! is_user_logged_in() ) {
        wp_redirect( home_url( '/maintenance.php' ) );
        exit();
    }
}

function is_admin_user_logged_in() {
    $user = wp_get_current_user();
    return $user && array_intersect( $user->roles, array( 'administrator' ) );
}
```

### Add Product Tag @ WC Products Admin Dashboard
```php
//Add Product Tag @ WC Products Admin Dashboard
add_filter( 'woocommerce_product_filters', 'eg_filter_by_custom_taxonomy_dashboard_products' );
function eg_filter_by_custom_taxonomy_dashboard_products( $output ) {
  
   global $wp_query;
 
   $output .= wc_product_dropdown_categories( array(
    'show_option_none' => 'Φιλτράρισμα με tag',
    'taxonomy' => 'product_tag',
    'name' => 'product_tag',
    'selected' => isset( $wp_query->query_vars['product_tag'] ) ? $wp_query->query_vars['product_tag'] : '',
   ) );
  
   return $output;
 }
```
### Add Input Search To Admin Bar
```
//Add Search To Admin Bar
add_action('admin_bar_menu', 'boatparts_admin_bar_form');
function boatparts_admin_bar_form() {
  global $wp_admin_bar;

  $search_query = '';
  if ( $_GET['post_type'] == 'product' ) {
    $search_query = $_GET['s'];
  }

  $wp_admin_bar->add_menu(array(
    'id' => 'boatparts_admin_bar_form',
    'parent' => 'top-secondary',
    'title' => '<form method="get" action="'.get_site_url().'/wp-admin/edit.php?post_type=product">
      <input name="s" type="text" value="' . $search_query . '" style="height:20px;margin:5px 0;line-height:1em;"/> 
      <input type="submit" style="padding:3px 7px;line-height:1" value="Search Products"/> 
      <input name="post_type" value="product" type="hidden">
    </form>'
  ));
}
```

## Extra
### Add snow with custom script at footer
```
<?php 
// Snow
add_action( 'wp_footer', function () { if (!is_admin()) { ?>

<style>
canvas#canvas{
  position: absolute;
  left: 0;
  top: 0;
  z-index: 99;
  pointer-events: none;
  width: 100%;
}
</style>

<script>
document.addEventListener("DOMContentLoaded", function(){
setTimeout(function() {
var canv = `<canvas id="canvas"></canvas>`;
document.body.insertAdjacentHTML("beforeend", canv);    
var body = document.body,
htmlElement = document.documentElement;
var height = body.offsetHeight ;
document.querySelector("#canvas").style.height = height + "px"
function startAnimation() {
  const CANVAS_WIDTH = window.innerWidth;
  const CANVAS_HEIGHT = height;
  const MIN = 0;
  const MAX = CANVAS_WIDTH;
  const canvas = document.querySelector("#canvas");
  const ctx = canvas.getContext("2d");
  canvas.width = CANVAS_WIDTH;
  canvas.height = CANVAS_HEIGHT;
  function clamp(number, min = MIN, max = MAX) {
    return Math.max(min, Math.min(number, max));
  }
  function random(factor = 1) {
    return Math.random() * factor;
  }
  function degreeToRadian(deg) {
    return deg * (Math.PI / 180);
  }
  class Circle {
    radius = 0;
    x = 0;
    y = 0;
    vx = 0;
    vy = 0;
    constructor(ctx) {
      this.ctx = ctx;
      this.reset();
    }
    draw() {
      this.ctx.beginPath();
      this.ctx.fillStyle = "rgba(255,255,255,0.8)";
      this.ctx.arc(this.x, this.y, this.radius, 0, 2 * Math.PI);
      this.ctx.fill();
      this.ctx.closePath();
    }
    reset() {
      this.radius = random(2.8);
      this.x = random(CANVAS_WIDTH);
      this.y = this.y ? 0 : random(CANVAS_HEIGHT);
      this.vx = clamp((Math.random() - 0.5) * 0.4, -0.4, 0.4);
      this.vy = clamp(random(1.5), 0.1, 0.8) * this.radius * 0.5;
    }
  }
  let circles = [];
  for (let i = 0; i < 300; i++) {
    circles.push(new Circle(ctx));
  }
  function clearCanvas() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  }
  let canvasOffset = {
    x0: ctx.canvas.offsetLeft,
    y0: ctx.canvas.offsetTop,
    x1: ctx.canvas.offsetLeft + ctx.canvas.width,
    y1: ctx.canvas.offsetTop + ctx.canvas.height
  };
  function animate() {
    clearCanvas();
    circles.forEach((e) => {
      if (
        e.x <= canvasOffset.x0 ||
        e.x >= canvasOffset.x1 ||
        e.y <= canvasOffset.y0 ||
        e.y >= canvasOffset.y1
      ) {
        e.reset();
      }
      e.x = e.x + e.vx;
      e.y = e.y + e.vy;
      e.draw();
    });
    requestAnimationFrame(animate);
  }
  animate();
}
startAnimation();
window.addEventListener("resize", startAnimation);
}, 500);	
});
</script>

<?php } });

// The End . . .

```

