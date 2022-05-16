#  MOD 04/05/2022
- Mostrar las categorías hermanas en la página de categoría [Prestashop 1.7]
- Link de video [https://www.youtube.com/watch?v=xbdNglSZChs](https://www.youtube.com/watch?v=xbdNglSZChs)

[![Alt text](https://img.youtube.com/vi/xbdNglSZChs/0.jpg)](https://www.youtube.com/watch?v=xbdNglSZChs)

Siga los siguientes pasos de acontinuación:

## Proceso
### 1: Paso
- Ingresar a tu proyecto y buscar el archivo **Category.php** en la ruta
  **/classes/** y agregar el siguiente código despues de la funcion **getSubCategories()**
  
```
        public function getParentCategories($idLang, $active = true)
    {
        $sqlGroupsWhere = '';
        $sqlGroupsJoin = '';
        if (Group::isFeatureActive()) {
            $sqlGroupsJoin = 'LEFT JOIN `' . _DB_PREFIX_ . 'category_group` cg ON (cg.`id_category` = c.`id_category`)';
            $groups = FrontController::getCurrentCustomerGroups();
            $sqlGroupsWhere = 'AND cg.`id_group` ' . (count($groups) ? 'IN (' . implode(',', $groups) . ')' : '=' . (int) Configuration::get('PS_UNIDENTIFIED_GROUP'));
        }
        $result = Db::getInstance(_PS_USE_SQL_SLAVE_)->executeS('
		SELECT c.*, cl.`id_lang`, cl.`name`, cl.`description`, cl.`link_rewrite`, cl.`meta_title`, cl.`meta_keywords`, cl.`meta_description`
		FROM `' . _DB_PREFIX_ . 'category` c
		' . Shop::addSqlAssociation('category', 'c') . '
		LEFT JOIN `' . _DB_PREFIX_ . 'category_lang` cl ON (c.`id_category` = cl.`id_category` AND `id_lang` = ' . (int) $idLang . ' ' . Shop::addSqlRestrictionOnLang('cl') . ')
		' . $sqlGroupsJoin . '
		WHERE `id_parent` = ' . (int) $this->id_parent . ' AND c.`id_category` <> '.(int) $this->id.'
		' . ($active ? 'AND `active` = 1' : '') . '
		' . $sqlGroupsWhere . '
		GROUP BY c.`id_category`
		ORDER BY `level_depth` ASC, category_shop.`position` ASC');
        foreach ($result as &$row) {
            $row['id_image'] = Tools::file_exists_cache($this->image_dir . $row['id_category'] . '.jpg') ? (int) $row['id_category'] : Language::getIsoById($idLang) . '-default';
            $row['legend'] = 'no picture';
        }
        return $result;
    }
   ``` 
### 2: Paso
- Ingresar a tu proyecto y buscar el archivo **CategoryController.php** en la ruta
  **/controllers/front/listing/** y agregar el siguiente código despues de la funcion **getTemplateVarSubCategories()**
  
  ```
      protected function getTemplateVarParentCategories()
    {
        return array_map(function (array $category) {
            $object = new Category(
                $category['id_category'],
                $this->context->language->id
            );
            $category['image'] = $this->getImage(
                $object,
                $object->id_image
            );
            $category['url'] = $this->context->link->getCategoryLink(
                $category['id_category'],
                $category['link_rewrite']
            );
            return $category;
        }, $this->category->getParentCategories($this->context->language->id));
    }
  ```
 - En el mismo archivo en la funcion **init()** agregar el siguiente codigo
    ```
    $this->context->smarty->assign([
            'parentcategories' => $this->getTemplateVarParentCategories(),
        ]);
    ```
### 3: Paso
- Ahora agregar en el tpl de tu plantilla y buscar la ruta **/themes/[tu-theme]/templates/catalog/listing/product-list.tpl** y agregar el siguiente código en la seccion que deas que aparesca 

  ```
    {block name='product_list_parentcategory'}
      <div id="parentcategories">
        <p class="parentcategory-heading">{l s='Parentcategories' d='Shop.Theme.Catalog'}</p>
        <ul class="clearfix" style="display: flex; gap: 10px;">
        {foreach from=$parentcategories item=parentcategory}
          <li>
            <a href="{$parentcategory.url|escape:'html':'UTF-8'}" title="{$parentcategory.name|escape:'html':'UTF-8'}" class="img">
              <img class="replace-2x" src="{$parentcategory.image.medium.url|escape:'html':'UTF-8'}" alt="{$parentcategory.name|escape:'html':'UTF-8'}" />
            </a>
            <h5>
              <a class="parentcategory-name" href="{$parentcategory.url|escape:'html':'UTF-8'}" >{$parentcategory.name|truncate:25:'...'|escape:'html':'UTF-8'}</a>
            </h5>
          </li>
        {/foreach}
        </ul>
      </div>
    {/block}
    ```
    
## Contribución
1. Luis Huaymana
2. [Prestademia](https://www.youtube.com/c/prestademia) : [https://www.youtube.com/c/prestademia](https://www.youtube.com/c/prestademia)
