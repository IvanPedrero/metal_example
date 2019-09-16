# README 
## Render Pipeline & renderizar objetos primitivos con *Swift* usando el framework **Metal**

# Indice 
- [Render Pipelines](#RenderPipelines)
    - [Funcionamiento](#Funcionamiento)
- [Programa básico](#SimpleMetal)
- [VideoTutorial](https://www.youtube.com/watch?v=jj2539vJ_Kc&feature=youtu.be)


## **RenderPipelines**

Un **Render Pipeline**  procesa comandos de dibujo y escribe datos con el objetivos de renderizado.

Las 3 principales fases del **Render Pipeline** son las funciones de *vértices, rasterización* y las funciones de *fragmentos*. 

![alt text](https://github.com/IvanPedrero/metal_example/blob/master/git_img/gitimg1.jpeg)
___

Vamos a escribir las funciones vértices y fragmentos para renderizar un triángulo simple en la pantalla. 
 
![alt text](https://github.com/IvanPedrero/metal_example/blob/master/git_img/gitimg2.jpeg)

El triangulo inherentemente cuenta con 3 vértices, los cuales tendrán coordenadas fijas para este READM
___

### **Funcionamiento**
Nuestra función de vertices (`Vertex`) tiene que mandar las coordenas de los respectivos vertices a la funcion `Fragment`. 

Una Función de Fragmento (`Fragment`) procesa la  iformación del rasterizador y calcula los valores de salida para cada uno de los objetivos del rasterizador. 

![alt text](https://github.com/IvanPedrero/metal_example/blob/master/git_img/gitimg3.jpeg)

Solo se renderizan los fragmentos cuyos centros de pixel estan dentro del triángulo. 

Toda esta información de vértices se pasa a través de buffers. En especifico de un *MTLBuffer*

![alt text](https://github.com/IvanPedrero/metal_example/blob/master/git_img/gitimg4.jpeg)

Se usan los *MTLBuffer* para datos al *Sahder* de los vèrtices. Para este proceso es recomendable utilizar la __GPU__ 

Para las operaciones de Commit y Dibujo necesitamos un *Command Queue*.
 
![alt text](https://github.com/IvanPedrero/metal_example/blob/master/git_img/gitimg5.jpeg)

Esto para hacer un *Queue* con el *Buffer* de vértices para poder  desplegarlos en la pantalla. 
___
## Programa

### SimpleMetal 

1. Iniciamos Xcode 
2. Damos clic en *"Create a new Xcode project"*
3. Seleccionamos la pestaña de *"macOS"*
4. En la seccion de Application seleccionamos la opción **Cocoa App** y damos clic en *"Next"*
5. Nombramos el proyecto, damos clic nuevamente en *"Next"* y creamos el proyecto con *"Create"*
6. Nos dirigimos al **Main.storyboard** y eliminamos el **ViewContorller** que etsa asociado a esta ventana  
7.  Damos clic drecho, seleccionamos *"New file..."* y seleccionamos nuevamente la opcion de *"Cocoa File"* para poder hacer nuestra propia clase y por ende controlar la vista 
8. Nombramos la clase (eg. MetalView), clic en *"Next"* y luego en *"Create"*
9. En este archivo qe sera nuestro controlador de vista principal cambiaremos:

```Swift
    import Cocoa
```

por 

```Swift
    import MetalKi
```


10. Posteriormente se inicia la clase:

```Swift
required init(coder: NSCoder) {
   super.init(coder: NSCoder)
}
```
11. Se definen nuestras variables `MTLCommandQueue` y `MTLRenderPipelineState`

```Swift
    var commandQueue: MTLCommandQueue !
    var renderPipelineState: MTLRenderPipelineState !
```
12. Deinimos la intefaz a la GPU en la variable `device`, la pintamos de verde (arbitrario) y asignamos el comando queue a la  interfaz  de la GPU

```Swift
required init(coder: NSCoder) {
   super.init(coder: NSCoder)

    self.device =  MTLCreateSystemDefaultDevice()
    self.clearcolor =  MTLClearColor(red: 0.45, green: 0.73, blue: 0.35, alpha: 1.0)
    self.colorPixelFormat = .bgra8Unorm
    self.commandQueue = device?.makeCommandQueue()
}
```
13. Iniciamos el **Render Pipeline State** para poder agregar las funciones `Vertex` y `Fragment`al Pipeline. 

```Swift
    func createRenderPipelineState(){
        let library = device?.makeDefaultLibrary()
        let vertexFunction = library?.makeFunction(name: "")
        let fragmentFunction = library?.makeFunction(name: "")
    }
```
14. El parametro que se debe mandar es el nombre de las funciones, etas sera definidas en otro archivo. pero primero se crea el **descriptor del Render Pipeline**

![i1](img_git/gitimg1)

```Swift
    func createRenderPipelineState(){
        let library = device?.makeDefaultLibrary()
        let vertexFunction = library?.makeFunction(name: "")
        let fragmentFunction = library?.makeFunction(name: "")

        let renderPipelineDescriptor =MTLRenderPipelineDescriptor()
        renderPipelineDescriptor.colorAttachments[0].pixelFormat = .bgra8Unorm
        renderPipelineDescriptor.vertexFucntion =  vertexFucntion
        renderPipelineDescriptor.fragmentFucntion =  fragmentFucntion

        do{
            renderPipelineDrescriptor = try device?.makeRenderPipelineState(drecriptor: renderPipelineDescriptor)
        }catch let error as NSError{
            print(error)
        }
    }
```

15. Despues creamos las funciones de `Vertex` y `Fragment` mediante un nuevo archivo de **Metal**, damos clic derecho *"New File..."* seleccionamos *"Metal File"*, clic en *"Next"* nombramos el archivo (eg. Shader) y finalmente *"Create"*. Dentro del archivo de **Metal**:

```Swift
    vertex float4 basic_vertex_shader(device float3 *vetices [[buffer(0)]],uint vertexID [[ vertex_id]]){
        return float4(vertices[vertexID],1):
    }
    fragment half4 basic_fragment_shader(){
        return half4(1):
    }
```

16. En el "Metal View" hay que crear un **Render Pipeline** y el *Buffer* en la funcion de inicio y agregar los nombres a de las funciones



```Swift
required init(coder: NSCoder) {
   super.init(coder: NSCoder)

    self.device =  MTLCreateSystemDefaultDevice()
    self.clearcolor =  MTLClearColor(red: 0.45, green: 0.73, blue: 0.35, alpha: 1.0)
    self.colorPixelFormat = .bgra8Unorm
    self.commandQueue = device?.makeCommandQueue()
 /// Se agrega aqui! (1)
    createRenderPipelineState()
    createBuffer()
}

    func createRenderPipelineState(){
        let library = device?.makeDefaultLibrary()
        /// Y aqui! (2)
        let vertexFunction = library?.makeFunction(name: "basic_vertex_shader")
        let fragmentFunction = library?.makeFunction(name: "basic_fragment_shader")
        /// . 
        let renderPipelineDescriptor =MTLRenderPipelineDescriptor()
        renderPipelineDescriptor.colorAttachments[0].pixelFormat = .bgra8Unorm
        renderPipelineDescriptor.vertexFucntion =  vertexFucntion
        renderPipelineDescriptor.fragmentFucntion =  fragmentFucntion

        do{
            renderPipelineDrescriptor = try device?.makeRenderPipelineState(drecriptor: renderPipelineDescriptor)
        }catch let error as NSError{
            print(error)
        }
    }
```
17. Definimos nuestros vertices y se argega la funcion para el *Buffer*

![i2](img_git/gitimg2)

```Swift
    var commandQueue: MTLCommandQueue !
    var renderPipelineState: MTLRenderPipelineState !
    /// modificaciones AQUI
    let vertices: [float3] = [
        float3(0, 1, 0),      //top middle
        float3(-1, -1, 0),    //bottom right
        float3(1, -1, 0)      //bottom left
    ]
    var vertexBuffer: MTLBuffer!
    ///.

required init(coder: NSCoder) {
   super.init(coder: NSCoder)

    self.device =  MTLCreateSystemDefaultDevice()
    self.clearcolor =  MTLClearColor(red: 0.45, green: 0.73, blue: 0.35, alpha: 1.0)
    self.colorPixelFormat = .bgra8Unorm
    self.commandQueue = device?.makeCommandQueue()

    createRenderPipelineState()
}
    /// modificacion AQUI
    func createBuffer(){
        // Nota al buffer tiene que ser del tamaño del tipo de dato que procese multiplicado por el numero de vertices, para este particular caso serian 16 bytes 
        vertexBuffer = device?.makeBuffer(bytes: vertices, lenght: MemoryLayout<float3>.stride * vertices.count, options: [])
    }
    ///.
    func createRenderPipelineState(){
        let library = device?.makeDefaultLibrary()
        let vertexFunction = library?.makeFunction(name: "basic_vertex_shader")
        let fragmentFunction = library?.makeFunction(name: "basic_fragment_shader")
        let renderPipelineDescriptor =MTLRenderPipelineDescriptor()
        renderPipelineDescriptor.colorAttachments[0].pixelFormat = .bgra8Unorm
        renderPipelineDescriptor.vertexFucntion =  vertexFucntion
        renderPipelineDescriptor.fragmentFucntion =  fragmentFucntion

        do{
            renderPipelineDrescriptor = try device?.makeRenderPipelineState(drecriptor: renderPipelineDescriptor)
        }catch let error as NSError{
            print(error)
        }
    }
```

18. Finalmente podemos hacer las operaciones de dibujo para la imagen renderizada mediante **Render Command Encoder**,se le indica que tiene que sacar la información, finalizar el proceso de codificación y confrimar el comando Buffer 

```Swift
    override func draw(_ dirtyRect: NSRect){
        guard let drawable = self.currentDrawable, let renderPassDescriptor =  self.currentRenderPassDescriptor else {return}
        let commandBuffer =  commandQueue.makeCommandBuffer()
        let renderCommandEncoder = commandBuffer?.makeRenderCommandEncoder(descriptor: renderPassDescriptor)
        renderCommandEncoder?.setRenderPipelineState(renderPipelineState)
        //indicación de salida de información 
        renderCommandEncoder?.setVertexBuffer(vertexBuffer, offset: 0, index: 0)
        renderCommandEncoder?.drawPrimitives(type: .triangule, vertexStart: 0, vertexCount: vertices.count)
        /// finalizar codificación y confirmar
        renderCommandEncoder?.endEncoding()
        commandBuffer?.present(drawable)
        commandBuffer?commit()
    }
```

19. En el Main.storyboard seleccionamos el *View Controller* y en la pestaña de *Custom Class* se cambia la clase por la anteriormente creada **MetalView*
20. Se corre el programa y se obesrva en el simulador. 
___

