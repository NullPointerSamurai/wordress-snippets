## Automatically delete WC images from Server after deleting a product

```
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

## Payment & SHipping Snippets
### Hide shipping rates when free shipping is available, but keep "Local pickup"
```
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
### Hide specific payment method based on total weight in Woocommerce
```
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
```
_metaseo_metatitle
_metaseo_metadesc
_metaseo_metakeywords
_metaseo_metaspecific_keywords

we_skroutzxml_custom_availability

```

## Administrator Dashboard
### Add Product Tag @ WC Products Admin Dashboard
```
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
