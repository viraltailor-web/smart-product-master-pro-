<?php
/*
Plugin Name: Smart Product Master Pro
Description: BLUE EDITION: Professional Blue Buttons, Clear Filters, and Hover Effects.
Version: 35.0
Author: Viral Tailor
*/

if ( ! defined( 'ABSPATH' ) ) exit;

// 1. REGISTER CPT
add_action( 'init', function() {
    register_post_type( 'smart_product', [
        'public' => true,
        'label'  => 'Smart Products',
        'menu_icon' => 'dashicons-cart',
        'supports' => ['title', 'editor', 'thumbnail'],
        'has_archive' => true,
        'rewrite' => ['slug' => 'product-item'],
        'show_in_rest' => true
    ]);
});

// 2. AJAX ENGINE (Blue Styling & Hover)
add_action('wp_ajax_vt_v35_action', 'vt_v35_handler');
add_action('wp_ajax_nopriv_vt_v35_action', 'vt_v35_handler');

function vt_v35_handler() {
    $paged = !empty($_POST['pg']) ? intval($_POST['pg']) : 1;
    $args = [
        'post_type' => 'smart_product',
        'posts_per_page' => 8,
        'paged' => $paged,
        's' => sanitize_text_field($_POST['kw'] ?? ''),
        'meta_query' => ['relation' => 'AND']
    ];

    if(!empty($_POST['pr'])) $args['meta_query'][] = ['key'=>'_price','value'=>$_POST['pr'],'compare'=>'<=','type'=>'NUMERIC'];
    if(!empty($_POST['rt'])) $args['meta_query'][] = ['key'=>'_rating','value'=>$_POST['rt'],'compare'=>'='];
    if(!empty($_POST['st'])) $args['meta_query'][] = ['key'=>'_stock','value'=>$_POST['st'],'compare'=>'='];

    $q = new WP_Query($args);
    if($q->have_posts()) {
        // Hover CSS for Cards and Blue Buttons
        echo "<style>
            .vt-card:hover { transform: translateY(-5px); box-shadow: 0 8px 25px rgba(0,0,0,0.1); border-color:#0056b3 !important; }
            .btn-blue { background:#007bff; border:1px solid #0056b3; color:#fff; transition: 0.3s; }
            .btn-blue:hover { background:#0056b3; border-color:#004085; }
        </style>";
        
        echo "<div style='display:grid; grid-template-columns:repeat(4, 1fr); gap:25px; width:100%;'>";
        while($q->have_posts()) { $q->the_post();
            $pr = get_post_meta(get_the_ID(), '_price', true);
            $rt = get_post_meta(get_the_ID(), '_rating', true);
            $st = get_post_meta(get_the_ID(), '_stock', true);
            $im = get_post_meta(get_the_ID(), '_v_img', true);
            ?>
            <div class="vt-card" style="border:1px solid #ddd; padding:20px; background:#fff; border-radius:12px; display:flex; flex-direction:column; height:460px; transition: all 0.3s ease;">
                <div style="flex-grow:1; text-align:center;">
                    <img src="<?php echo $im; ?>" style="width:100%; height:180px; object-fit:cover; border-radius:8px; margin-bottom:15px;">
                    <h5 style="margin:0 0 10px 0; font-size:16px; height:42px; overflow:hidden;">
                        <a href="<?php the_permalink(); ?>" style="text-decoration:none; color:#222; font-weight:700;"><?php the_title(); ?></a>
                    </h5>
                    <div style="color:#ffa41c; margin-bottom:10px;"><?php echo str_repeat('★', (int)$rt) . str_repeat('☆', 5-(int)$rt); ?></div>
                    <p style="color:#B12704; font-weight:bold; font-size:22px; margin:5px 0;">$<?php echo $pr; ?></p>
                    <p style="font-size:13px; font-weight:bold; color:<?php echo ($st=='In-stock'?'green':'red'); ?>;"><?php echo $st; ?></p>
                </div>
                <button onclick="alert('Success: Added to Cart!')" class="btn-blue" style="padding:12px; cursor:pointer; width:100%; border-radius:6px; font-weight:bold; font-size:14px;">Add to Cart</button>
            </div>
            <?php
        }
        echo "</div>";
        
        echo "<div style='margin-top:50px; text-align:center; width:100%;'>";
        for($i=1; $i<=$q->max_num_pages; $i++) {
            $active = ($i==$paged) ? 'background:#232f3e; color:#fff;' : 'background:#fff; color:#232f3e; border:1px solid #232f3e;';
            echo "<button onclick='v35Search($i)' style='padding:12px 22px; margin:0 8px; cursor:pointer; border-radius:6px; font-weight:bold; $active'>$i</button>";
        }
        echo "</div>";
    } else { echo "No products found."; }
    wp_die();
}

// 3. MAIN UI (Clear Filter & Blue Style)
add_shortcode('product_filter', function() {
    ob_start(); ?>
    <div style="width:100%; max-width:1400px; margin:0 auto; display:flex; gap:40px; font-family:sans-serif; align-items:flex-start;">
        <aside style="width:280px; min-width:280px; background:#f9f9f9; padding:25px; border-radius:12px; border:1px solid #eee;">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:20px;">
                <h3 style="margin:0; font-size:18px;">Filters</h3>
                <span onclick="clearFilters()" style="font-size:12px; color:#007bff; cursor:pointer; text-decoration:underline; font-weight:bold;">Clear All</span>
            </div>
            
            <label style="font-weight:bold; font-size:13px;">Max Price:</label>
            <input type="number" id="v35_pr" style="width:100%; padding:12px; margin:8px 0 20px 0; border:1px solid #ccc; border-radius:6px;">
            
            <label style="font-weight:bold; font-size:13px;">Rating:</label>
            <select id="v35_rt" style="width:100%; padding:12px; margin:8px 0 20px 0; border:1px solid #ccc; border-radius:6px;">
                <option value="">Any</option><option value="5">5 Stars</option><option value="4">4 Stars</option>
            </select>
            
            <label style="font-weight:bold; font-size:13px;">Availability:</label>
            <div style="margin:10px 0;"><input type="radio" name="st" class="v35_st" value="In-stock"> In-stock</div>
            <div style="margin-bottom:10px;"><input type="radio" name="st" class="v35_st" value="Out of stock"> Out of stock</div>
            <div style="margin-bottom:25px;"><input type="radio" name="st" class="v35_st" value="" id="st_all" checked> All Items</div>
            
            <button onclick="v35Search(1)" style="width:100%; background:#232f3e; color:#fff; padding:15px; border:none; border-radius:8px; font-weight:bold; cursor:pointer;">UPDATE RESULTS</button>
        </aside>

        <div style="flex:1;">
            <input type="text" id="v35_kw" onkeyup="v35Search(1)" placeholder="Search gear..." style="width:100%; padding:18px; margin-bottom:35px; border:1px solid #ddd; border-radius:10px;">
            <div id="v35_results">Loading...</div>
        </div>
    </div>

    <script>
    function clearFilters() {
        document.getElementById('v35_kw').value = '';
        document.getElementById('v35_pr').value = '';
        document.getElementById('v35_rt').value = '';
        document.getElementById('st_all').checked = true;
        v35Search(1);
    }
    function v35Search(page = 1) {
        let box = document.getElementById('v35_results');
        let st = ""; document.querySelectorAll('.v35_st').forEach(r => { if(r.checked) st = r.value; });
        let p = new URLSearchParams();
        p.append('action', 'vt_v35_action');
        p.append('kw', document.getElementById('v35_kw').value);
        p.append('pr', document.getElementById('v35_pr').value);
        p.append('rt', document.getElementById('v35_rt').value);
        p.append('st', st);
        p.append('pg', page);
        fetch('<?php echo admin_url('admin-ajax.php'); ?>', { method:'POST', body:p }).then(r=>r.text()).then(h=>{box.innerHTML=h;});
    }
    document.addEventListener('DOMContentLoaded', () => v35Search(1));
    </script>
    <?php return ob_get_clean();
});

// 4. DETAIL PAGE (Blue Button Fix)
add_filter('the_content', function($content) {
    if (is_singular('smart_product')) {
        $id = get_the_ID();
        $pr = get_post_meta($id, '_price', true);
        $rt = get_post_meta($id, '_rating', true);
        $st = get_post_meta($id, '_stock', true);
        $im = get_post_meta($id, '_v_img', true);
        return "
        <div style='max-width:950px; margin:40px auto; display:flex; gap:40px; padding:35px; background:#fff; border:1px solid #eee; border-radius:15px; box-shadow:0 15px 40px rgba(0,0,0,0.1); font-family:sans-serif;'>
            <div style='flex:1;'><img src='$im' style='width:100%; border-radius:12px;'></div>
            <div style='flex:1;'>
                <h1 style='margin:0; font-size:32px;'>".get_the_title()."</h1>
                <div style='color:#ffa41c; font-size:22px; margin:15px 0;'>".str_repeat('★', (int)$rt)."</div>
                <p style='color:#B12704; font-size:42px; font-weight:bold; margin:15px 0;'>$$pr.00</p>
                <p style='font-size:18px;'>Availability: <span style='font-weight:bold; color:".($st=='In-stock'?'green':'red').";'>$st</span></p>
                <div style='padding:20px; background:#f9f9f9; border-radius:10px; margin:25px 0; border-left:5px solid #007bff;'>$content</div>
                <button onclick=\"alert('Success: Added to Cart!')\" style='background:#007bff; color:#fff; border:none; padding:20px; width:100%; border-radius:10px; font-weight:bold; font-size:20px; cursor:pointer;'>Add to Cart</button>
            </div>
        </div>";
    }
    return $content;
});
