> [!Note]
> This is a custom plugin developed by E-Grow, designed to enhance the functionality of WooCommerce with essential features.

```php
/**
 * Plugin Name: Fundamentals Functions
 * Description: This plugin offers a comprehensive array of functions tailored to meet the foundational needs of developers, data analysts, and system administrators alike.
 * Version: 1.0
 * Author: E-Grow
 * Author URI: https://www.e-grow.gr
 * Version: 1.0
 * Requires at least: 5
 * Requires PHP: 7.4
 */

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


// Hook into WordPress
add_action('woocommerce_loaded', 'eg_custom_category_thumbnail_init');
function eg_custom_category_thumbnail_init() {
    // Check if WooCommerce is active
    if (class_exists('WooCommerce')) {
        // Hook into the WooCommerce action
        add_action('woocommerce_before_subcategory_title', 'eg_replace_category_thumbnail', 10);
    }
}

// Declare functions
function eg_replace_category_thumbnail($category)
{
	 error_log('Function is running');
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
