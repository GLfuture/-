# Nginx模块的编译

### nginx模块有三个需要注意的地方

1.初始化的时候(进程刚开始启动的时候)

2.解析conf文件的时候

3.请求处理的流程

## 过滤器模块

1.在模块源文件目录下(/home/gong/src/nginx/module)创建config文件并编写config

```c++
ngx_addon_name=ngx_http_zry_filter_module//对应编写的ngx_module_t的变量名
HTTP_FILTER_MODULES="$HTTP_FILTER_MODULES ngx_http_zry_filter_module"//路径填过滤器源文件的路径
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_filter_module.c"
```

2.重新编译生成nginx

添加命令

```shell
--add-module=/home/gong/src/nginx/ngx_http_filter_module
```

### module实现

```c++
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>

typedef struct {
	ngx_flag_t enable;
} ngx_http_filter_conf_t;
/*
header
 */
static ngx_str_t prefix = ngx_string("<h2>King </h2>");
static ngx_http_output_header_filter_pt  ngx_http_next_header_filter;
static ngx_http_output_body_filter_pt    ngx_http_next_body_filter;
ngx_int_t ngx_http_zry_header_filter(ngx_http_request_t *r) {
	if (r->headers_out.status != NGX_HTTP_OK) {
		return ngx_http_next_header_filter(r);
	}
	//r->headers_out.content_type.len == sizeof("text/html")
	r->headers_out.content_length_n += prefix.len;
	return ngx_http_next_header_filter(r);
}

ngx_int_t ngx_http_zry_body_filter(ngx_http_request_t *r, ngx_chain_t *chain) {

	ngx_buf_t *b = ngx_create_temp_buf(r->pool, prefix.len);
	b->start = b->pos = prefix.data;
	b->last = b->pos + prefix.len;

	ngx_chain_t *c1 = ngx_alloc_chain_link(r->pool);
	c1->buf = b;
	c1->next = chain;

	return ngx_http_next_body_filter(r, c1);

}
ngx_int_t   ngx_http_zry_filter_init(ngx_conf_t *cf) {

	ngx_http_next_header_filter = ngx_http_top_header_filter;
	ngx_http_top_header_filter = ngx_http_zry_header_filter;

	ngx_http_next_body_filter = ngx_http_top_body_filter;
	ngx_http_top_body_filter = ngx_http_zry_body_filter;

	return NGX_OK;
}
void *ngx_http_zry_filter_create_loc_conf(ngx_conf_t *cf) {

	ngx_http_filter_conf_t *conf = ngx_palloc(cf->pool, sizeof(ngx_http_filter_conf_t));
	if (conf == NULL) return NULL;

	conf->enable = NGX_CONF_UNSET;

	return conf;
}
char *ngx_http_zry_filter_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child) {

	ngx_http_filter_conf_t *prev = (ngx_http_filter_conf_t*)parent;
	ngx_http_filter_conf_t *next = (ngx_http_filter_conf_t*)child;

	//printf("ngx_http_zry_filter_merge_loc_conf: %d\n", next->enable);
	ngx_conf_merge_value(next->enable, prev->enable, 0);

	return NGX_CONF_OK;
}
/*
char *ngx_http_zry_filter_set_slot(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {
	char *p = conf;
	ngx_flag_t *flag = (p + cmd->offset);
	if (*flag == NGX_CONF_UNSET) {		
	}
	return NGX_CONF_OK;
}

/*
struct ngx_command_s {
    ngx_str_t             name;
    ngx_uint_t            type;
    char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
    ngx_uint_t            conf;
    ngx_uint_t            offset;
    void                 *post;
};
*/
// prefix on/off
ngx_command_t ngx_http_zry_filter_module_cmd[] = {
	{
		ngx_string("prefix"),
		NGX_HTTP_MAIN_CONF | NGX_HTTP_SRV_CONF | NGX_HTTP_LOC_CONF | NGX_CONF_FLAG,
		//ngx_http_zry_filter_set_slot,
		ngx_conf_set_flag_slot,
		NGX_HTTP_LOC_CONF_OFFSET,
		offsetof(ngx_http_filter_conf_t, enable),
		NULL,
	},
	ngx_null_command
};
// nginx -c conf/nginx.conf
// module 
static ngx_http_module_t ngx_http_zry_filter_module_ctx = {
	NULL,
	ngx_http_zry_filter_init,
	NULL,
	NULL,
	NULL,
	NULL,
	ngx_http_zry_filter_create_loc_conf,
	ngx_http_zry_filter_merge_loc_conf,

};
ngx_module_t ngx_http_zry_filter_module = {
	NGX_MODULE_V1,
	&ngx_http_zry_filter_module_ctx,
	ngx_http_zry_filter_module_cmd,
	NGX_HTTP_MODULE,
	NULL,
	NULL,
	NULL,
	NULL,
	NULL,
	NULL,
	NULL,
	NGX_MODULE_V1_PADDING
};
```

## handle模块

```c++
#include <ngx_http.h>
#include <ngx_config.h>
#include <ngx_core.h>

/*
#include <arpa/inet.h>
#include <netinet/in.h>
*/
#define ENABLE_RBTREE	1


static char *ngx_http_pagecount_set(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
static ngx_int_t ngx_http_pagecount_handler(ngx_http_request_t *r);
static ngx_int_t ngx_http_pagecount_init(ngx_conf_t *cf);
static void  *ngx_http_pagecount_create_location_conf(ngx_conf_t *cf);
static ngx_int_t ngx_http_pagecount_shm_init (ngx_shm_zone_t *zone, void *data);
static void ngx_http_pagecount_rbtree_insert_value(ngx_rbtree_node_t *temp,
        ngx_rbtree_node_t *node, ngx_rbtree_node_t *sentinel);


static ngx_command_t count_commands[] = {
	{
		ngx_string("count"),
		NGX_HTTP_LOC_CONF | NGX_CONF_NOARGS,
		ngx_http_pagecount_set,
		NGX_HTTP_LOC_CONF_OFFSET,
		0, NULL
	},
	ngx_null_command
};

static ngx_http_module_t count_ctx = {
	NULL,
	ngx_http_pagecount_init,
	
	NULL,
	NULL,

	NULL,
	NULL,

	ngx_http_pagecount_create_location_conf,
	NULL,
};

//ngx_http_count_module 
ngx_module_t ngx_http_pagecount_module = {
	NGX_MODULE_V1,
	&count_ctx,
	count_commands,
	NGX_HTTP_MODULE,
	NULL,
	NULL,
	NULL,
	NULL,
	NULL,
	NULL,
	NULL,
	NGX_MODULE_V1_PADDING
};

typedef struct {
	int count; //count
} ngx_http_pagecount_node_t;

typedef struct {

	ngx_rbtree_t rbtree;
	ngx_rbtree_node_t sentinel;
	
} ngx_http_pagecount_shm_t;

typedef struct
{
    ssize_t shmsize;
 
    ngx_slab_pool_t *shpool;
 
    ngx_http_pagecount_shm_t *sh;
} ngx_http_pagecount_conf_t;


ngx_int_t ngx_http_pagecount_shm_init (ngx_shm_zone_t *zone, void *data) {

	ngx_http_pagecount_conf_t *conf;
	ngx_http_pagecount_conf_t *oconf = data;

	conf = (ngx_http_pagecount_conf_t*)zone->data;
	if (oconf) {
		conf->sh = oconf->sh;
		conf->shpool = oconf->shpool;
		return NGX_OK;
	}

	//printf("ngx_http_pagecount_shm_init 0000\n");
	
	conf->shpool = (ngx_slab_pool_t*)zone->shm.addr;
	conf->sh = ngx_slab_alloc(conf->shpool, sizeof(ngx_http_pagecount_shm_t));
	if (conf->sh == NULL) {
		return NGX_ERROR;
	}

	conf->shpool->data = conf->sh;

	//printf("ngx_http_pagecount_shm_init 1111\n");
	
	ngx_rbtree_init(&conf->sh->rbtree, &conf->sh->sentinel, 
		ngx_http_pagecount_rbtree_insert_value);


	return NGX_OK;

}


static char *ngx_http_pagecount_set(ngx_conf_t *cf, ngx_command_t *cmd, void *conf) {

	ngx_shm_zone_t *shm_zone;
	ngx_str_t name = ngx_string("pagecount_slab_shm");

	ngx_http_pagecount_conf_t *mconf = (ngx_http_pagecount_conf_t*)conf;
	ngx_http_core_loc_conf_t *corecf;
	
	//ngx_log_error(NGX_LOG_EMERG, cf->log, ngx_errno, "ngx_http_pagecount_set000");
	
	mconf->shmsize = 1024*1024;
	
	shm_zone = ngx_shared_memory_add(cf, &name, mconf->shmsize, &ngx_http_pagecount_module);
	if (NULL == shm_zone) {
		return NGX_CONF_ERROR;
	}

	
	shm_zone->init = ngx_http_pagecount_shm_init;
	shm_zone->data = mconf;

	corecf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
	corecf->handler = ngx_http_pagecount_handler;

	return NGX_CONF_OK;
}

ngx_int_t   ngx_http_pagecount_init(ngx_conf_t *cf) {
#if 0
	ngx_http_core_loc_conf_t *cmcf = ngx_http_conf_get_module_main_conf(cf, ngx_http_core_module);

	ngx_http_handler_pt *h = ngx_array_push(cmcf->phases[NGX_HTTP_PREACCESS_PHASE].handlers);
	if (NULL == h) {
		return NGX_ERROR;
	}
	*h = ngx_http_pagecount_handler;
#elif 0

	ngx_log_error(NGX_LOG_NOTICE, cf->log, ngx_errno, "ngx_http_pagecount_init");

	ngx_http_core_loc_conf_t *corecf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
	corecf->handler = ngx_http_pagecount_handler;

	//return NGX_OK;
#endif

	return NGX_OK;
}

void  *ngx_http_pagecount_create_location_conf(ngx_conf_t *cf) {

	ngx_http_pagecount_conf_t *conf;
	
	
	conf = ngx_palloc(cf->pool, sizeof(ngx_http_pagecount_conf_t));
	if (NULL == conf) {
		return NULL;
	}

	conf->shmsize = 0;
	//ngx_log_error(NGX_LOG_EMERG, cf->log, ngx_errno, "ngx_http_pagecount_create_location_conf");

	// init conf data
	// ... 

	return conf;

}


static void
ngx_http_pagecount_rbtree_insert_value(ngx_rbtree_node_t *temp,
        ngx_rbtree_node_t *node, ngx_rbtree_node_t *sentinel)
{
   ngx_rbtree_node_t **p;
   //ngx_http_testslab_node_t *lrn, *lrnt;
 
    for (;;)
    {
        if (node->key < temp->key)
        {
            p = &temp->left;
        }
        else if (node->key > temp->key) {
           	p = &temp->right;
        }
        else
        {
          	return ;
        }
 
        if (*p == sentinel)
        {
            break;
        }
 
        temp = *p;
    }
 
    *p = node;
 
    node->parent = temp;
    node->left = sentinel;
    node->right = sentinel;
    ngx_rbt_red(node);
}

static ngx_int_t ngx_http_pagecount_lookup(ngx_http_request_t *r, ngx_http_pagecount_conf_t *conf, ngx_uint_t key) {

	ngx_rbtree_node_t *node, *sentinel;

	node = conf->sh->rbtree.root;
	sentinel = conf->sh->rbtree.sentinel;

	ngx_log_error(NGX_LOG_EMERG, r->connection->log, ngx_errno, " ngx_http_pagecount_lookup 111 --> %x\n", key);
	
	while (node != sentinel) {

		if (key < node->key) {
			node = node->left;
			continue;
		} else if (key > node->key) {
			node = node->right;
			continue;
		} else { // key == node
			node->data ++;
			return NGX_OK;
		}
	}
	
	ngx_log_error(NGX_LOG_EMERG, r->connection->log, ngx_errno, " ngx_http_pagecount_lookup 222 --> %x\n", key);
	
	// insert rbtree
	
	
	node = ngx_slab_alloc_locked(conf->shpool, sizeof(ngx_rbtree_node_t));
	if (NULL == node) {
		return NGX_ERROR;
	}

	node->key = key;
	node->data = 1;

	ngx_rbtree_insert(&conf->sh->rbtree, node);

	ngx_log_error(NGX_LOG_EMERG, r->connection->log, ngx_errno, " insert success\n");
	

	return NGX_OK;
}


static int ngx_encode_http_page_rb(ngx_http_pagecount_conf_t *conf, char *html) {

	sprintf(html, "<h1>Source Insight </h1>");
	strcat(html, "<h2>");

	//ngx_rbtree_traversal(&ngx_pv_tree, ngx_pv_tree.root, ngx_http_count_rbtree_iterator, html);
	ngx_rbtree_node_t *node = ngx_rbtree_min(conf->sh->rbtree.root, conf->sh->rbtree.sentinel);

	do {

		char str[INET_ADDRSTRLEN] = {0};
		char buffer[128] = {0};

		sprintf(buffer, "req from : %s, count: %d <br/>",
			inet_ntop(AF_INET, &node->key, str, sizeof(str)), node->data);

		strcat(html, buffer);

		node = ngx_rbtree_next(&conf->sh->rbtree, node);

	} while (node);
	

	strcat(html, "</h2>");

	return NGX_OK;
}



static ngx_int_t ngx_http_pagecount_handler(ngx_http_request_t *r) {


	u_char html[1024] = {0};
	int len = sizeof(html);
	
	ngx_rbtree_key_t key = 0;


	struct sockaddr_in *client_addr =  (struct sockaddr_in*)r->connection->sockaddr;
	

	ngx_http_pagecount_conf_t *conf = ngx_http_get_module_loc_conf(r, ngx_http_pagecount_module);
	key = (ngx_rbtree_key_t)client_addr->sin_addr.s_addr;

	ngx_log_error(NGX_LOG_EMERG, r->connection->log, ngx_errno, " ngx_http_pagecount_handler --> %x\n", key);

	ngx_shmtx_lock(&conf->shpool->mutex);
	ngx_http_pagecount_lookup(r, conf, key);	
	ngx_shmtx_unlock(&conf->shpool->mutex);
	
	ngx_encode_http_page_rb(conf, (char*)html);

	//header
	r->headers_out.status = 200;
	ngx_str_set(&r->headers_out.content_type, "text/html");
	ngx_http_send_header(r);

	//body
	ngx_buf_t *b = ngx_pcalloc(r->pool,  sizeof(ngx_buf_t));

	ngx_chain_t out;
	out.buf = b;
	out.next = NULL;

	b->pos = html;
	b->last = html+len;
	b->memory = 1;
	b->last_buf = 1;

	return ngx_http_output_filter(r, &out);
	
	
}

```







