#  MOD 15/05/2022
- Multiplos en pagina de productos y carrito de compras [Prestashop 1.7]
- Link de video [https://www.youtube.com/watch?v=SspCzjBAvTw](https://www.youtube.com/watch?v=SspCzjBAvTw)

[![Alt text](https://img.youtube.com/vi/SspCzjBAvTw/0.jpg)](https://www.youtube.com/watch?v=SspCzjBAvTw)

Siga los siguientes pasos de acontinuación:

## Proceso
### 1: Paso
- Ingresar a tu proyecto y buscar el archivo **product-add-to-cart.tpl** en la ruta
  **/themes/classic/template/catalog/_partials/** y agregar el siguiente código(artificio) despues del input con id= **"quantity_wanted"**
  
```
  <input type="hidden" id="check_step_min">
   ``` 
- Y en el mismo input agrega la class **quantity_wanted**.
### 2: Paso
- Ingresar a tu proyecto y buscar el archivo **custom.js** en la ruta
  **/themes/classic/assets/js/** y agregar el siguiente código
  
  ```
    $(function(){
       multiplyProductLoad();
       multiplyCart();

       prestashop.on('updatedProduct', function(){
           multiplyProduct();
       });

        prestashop.on('updateCart', function(){
            multiplyCart();
            let min_update = $('#quantity_wanted');
            let attr_update = min_update.attr('min');
            min_update.val(attr_update);
        });

     });

     function multiplyProduct(){
         let min_step_now = $("input.quantity_wanted").attr('min');
         let min_step_old = $("input#check_step_min").val();

         if(min_step_now != min_step_old){
             $("input.quantity_wanted").val(min_step_now);
             $("input.quantity_wanted").trigger("touchspin.updatesettings", {step: min_step_now});
             $("input#check_step_min").val(min_step_now);
         }
     }

     function multiplyProductLoad(){
         let min_step = $("input.quantity_wanted").attr('min');
         $("input#check_step_min").val(min_step);
         $("input.quantity_wanted").trigger("touchspin.updatesettings", {step: min_step});

     }

     function multiplyCart(){
         let inputs = $('input[name="product-quantity-spin"]');
         inputs.each(function(){
             let min_step_cart = $(this).attr('min');
             $(this).trigger("touchspin.updatesettings", {step: min_step_cart});
         });
     }
  ```
- Con todo lo anterior (paso 1 y paso 2) completamos la configuración para la ficha de producto y dejamos listo el js para el carrito de compras

### 3: Paso
- Ingresar a tu proyecto y buscar el archivo **Cart.php** en la ruta
  **/classes/** y busca la funcion llamada **function updateQty()** y copia todo esa funcion completa.

- Luego crea un override; si ya lo tienes creado agrega la funcion copiada dentro de la clase principal, como se muestra en el video y edita:

- Busca dentro de funcion la parte :

 ```
        if (!empty($id_product_attribute)) {
            $minimal_quantity = (int) Attribute::getAttributeMinimalQty($id_product_attribute);
        } else {
            $minimal_quantity = (int) $product->minimal_quantity;
        }
 ```

- Después de esta linea agrega lo siguiente:

  ```
     $verify_multiply = $this->getProductQuantity(
            $id_product,
            $id_product_attribute,
            (int) $id_customization,
            (int) $id_address_delivery
        );

        $min        = $minimal_quantity == 0 ? 1 : $minimal_quantity;
        $multiply   = $verify_multiply['quantity'] % $min;

        if($multiply != 0 ){
            $quantity = $min - $multiply;
        }
        else{
            $quantity = $min;
        }
  ```
  
### 4: Paso
- Ahora en este paso final agrega un attributo a un input en el archivo **cart-detailed-product-line.tpl** en la ruta
  **/themes/classic/template/checkout/_partials/**, busca la class= **"js-cart-line-product-quantity"** y agrega lo siguiente:
  
  ```
     min="{$product.minimal_quantity}"
  ```
  
- Y con eso terminariamos el carrito.

## Contribución
1. Luis Huaymana
2. [Prestademia](https://www.youtube.com/c/prestademia) : [https://www.youtube.com/c/prestademia](https://www.youtube.com/c/prestademia)
