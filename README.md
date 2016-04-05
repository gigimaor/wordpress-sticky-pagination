# wordpress-sticky-pagination
Inline sticky post to a list of posts that using pagination

The issue:
Wordpress WP_query and pagination, using sticky post from page 2 and more add this post to the list in the first page as additional post. it increasing the number of posts for the first page.

If you like to keep the number of posts on the first page as configured, but be able to add unlimited sticky posts to the top of the list.

First
Add sticky posts list before the main list (only for first page)

Second 
Offset the number of the sticky posts on a spesific page number for the main list (for all other pages then the first)


<code><pre>
function get_lastest_posts($ignore_sticky_posts=true){
  //get page number
	global $paged;
	$paged = ( get_query_var('page') ) ? get_query_var('page') : 1;
	$posts_per_page = 6;
	$args = array(
		'post_status'         =>  'publish',
		'posts_per_page'      => $posts_per_page,
		'orderby'             => 'date',
		'order'               => 'DESC',
		'paged'               => $paged,
		'ignore_sticky_posts' => 1,
	);
	$sticky = get_option( 'sticky_posts' );
	if($sticky)
		$sticky_count = sizeof($sticky);
	if($sticky_count > 0){
		if($paged == 1 ){
		  //Only for the first page prepare the sticky post first
			$args['post__in'] = $sticky;
			//Build sticky post query
			$local_sticky_wp_query = new WP_Query( $args );
		}
		else{
		  //For all other pages
		  //calculate the offset of the page, reduce the number of sticky posts
	    $page_offset = ( ($paged-1) * $posts_per_page ) - $sticky_count;
      //assign offset to main query (it will cancel the pagitation 
      //It will view only the numbers of post for a spesific page
	    $args['offset'] =  $page_offset ;
		}
		//reset main list
		//first page number of post reduce sticky posts
		$args['posts_per_page'] = $posts_per_page - $sticky_count;
		$args['post__in'] = null;
		$args['post__not_in'] = $sticky;
	}
	if($paged > 1 ){
	  //all other pages view number of posts
		$args['posts_per_page'] = $posts_per_page;
	}
  //Main query
	$local_wp_query = new WP_Query( $args );

	if($local_wp_query->post_count > 0){
		?><section class="latest-posts">
			<h2>Latest</h2>
		 	<div class="list-wrap">
    			<div class="list clearfix">
    				<div class="row"><?php
        			//view sticky posts
        			if($sticky_count > 0 && $paged == 1){
        				get_list_items($local_sticky_wp_query,false);
        			}
        			//view all other posts
        			get_list_items($local_wp_query,true);
				?>	</div>
				</div>
			</div>
		</section><?php
	}
}
</pre></code>
