//This functions allows you to switch menus depending on if a user is logged into your site or not and, if they are logged in, what they're user role is

function my_conditional_menus( $args = '' ) {
 
if( is_user_logged_in() && current_user_can('administrator') ) { 
    $args['menu'] = 'REPLACE-WITH-ADMIN-MENU-NAME'; //this menu will be shown ONLY to logged in admins
} elseif ( is_user_logged_in() && !current_user_can('administrator') ) { 
    $args['menu'] = 'REPLACE-WITH-NON-ADMIN-MENU-NAME'; //this menu will be shown to any logged in user that is NOT a site admin
	show_admin_bar(false); //removes the WP admin bar from the front end of the website
} else {
	$args['menu'] = 'REPLACE-WITH-MAIN-MENU-NAME'; //this menu will be shown to all logged out users
}
    return $args;
}
add_filter( 'wp_nav_menu_args', 'my_conditional_menus' );

//This function redirects to different pages based on user role

function my_login_redirect( $redirect_to, $request, $user ) {
    //is there a user to check?
    if ( isset( $user->roles ) && is_array( $user->roles ) ) {
        //check for admins
        if ( in_array( 'administrator', $user->roles ) ) {
            return home_url('wp-admin'); //replace wp-admin with the page you want to redirect admins to
            return $redirect_to;
        } else {
            return home_url('my-account'); //replace my-account with the page you want to redirect non-admins to
        }
    } else {
        return $redirect_to;
    }
}
add_filter( 'login_redirect', 'my_login_redirect', 10, 3 );

//this function removes the logout confirmation, but it requires that the logout URL be created as /wp-login?action=logout in a custom menu item under Appearance > Menus

function wpa_remove_menu_item( $items, $menu, $args ) {
    if ( is_admin() || ! is_user_logged_in() ) 
        return $items;
    foreach ( $items as $key => $item ) {
        if ( 'Login / Register' == $item->title ) 
            unset( $items[$key] );
        if ( 'Logout' == $item->title ) {
            $items[$key]->url = wp_logout_url();
        }
    }
    return $items;
}
add_filter( 'wp_get_nav_menu_items', 'wpa_remove_menu_item', 10, 3 );

//this function redirects the user to the home page after they log out

function redirect_after_logout(){
         wp_redirect( home_url() );
         exit();
}
add_action( 'wp_logout', 'redirect_after_logout');
