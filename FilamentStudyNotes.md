# Filament学习笔记  
## 渲染框架  
![渲染框架](https://github.com/zhangbo-0213/PictureRepository/blob/main/Filament渲染框架.png "渲染框架")
## Mac OSX 编译环境配置
1. 工具准备 
* Xcode 
* Cmake （[配置命令行工具](https://blog.csdn.net/weixin_41770169/article/details/85758091)）
* VScode (插件：CmakeTools,C/C++Extensions)  
2. 编译步骤  
* 针对M1配置launch.json [设置](https://blog.csdn.net/u014286840/article/details/115033198)
* cmake Build(command+shift+p)
* cmake SetTarget(command+shift+p)
* cmake Debug(command+shift+p)
* F5 调试  
## hellotriangle 学习  
```c++  
using namespace filament;
using utils::Entity;
using utils::EntityManager;

struct App {
    VertexBuffer* vb;
    IndexBuffer* ib;
    Material* mat;
    Camera* cam;
    Entity camera;
    Skybox* skybox;
    Entity renderable;
};

struct Vertex {
    filament::math::float2 position;
    uint32_t color;  //包含ARGB四个通道
};

static const Vertex TRIANGLE_VERTICES[3] = {
    {{1, 0}, 0xffff0000u},
    {{cos(M_PI * 2 / 3), sin(M_PI * 2 / 3)}, 0xff00ff00u},
    // {{0, -1}, 0xff00ff00u},
    {{cos(M_PI * 4 / 3), sin(M_PI * 4 / 3)}, 0xff0000ffu},
    // {{0,1}, 0xff0000ffu},
};

static constexpr uint16_t TRIANGLE_INDICES[3] = { 0, 1, 2 };

int main(int argc, char** argv) {
    Config config;
    config.title = "hellotriangle";

    App app;
    auto setup = [&app](Engine* engine, View* view, Scene* scene) {
        app.skybox = Skybox::Builder().color({0.25, 0.25, 0.25, 1.0}).build(*engine);
        scene->setSkybox(app.skybox);
        view->setPostProcessingEnabled(false);
        static_assert(sizeof(Vertex) == 12, "Strange vertex size.");
        //设置 VBO 顶点缓存对象 VertexBuffer::Builder 链式编程设置VBO属性
        app.vb = VertexBuffer::Builder()
                .vertexCount(3)
                .bufferCount(1)
                //attribute属性设置(属性索引，包含该属性的vertexBuffer的Index,属性的类型，在buffer中取数据的字节偏移量，该顶点所有属性的字节步长)
                //通过设置偏移量的方式，使用交错数组来节省verterBuffer的数量，这里通过交错数组使用一个verterBuffer保存两个顶点数据
                //POSITION属性为顶点的第一个属性，偏移长度为0；COLOR属性为第二个属性，偏移长度为2*float=8，两个属性共计字节长度为8+4=12
                .attribute(VertexAttribute::POSITION, 0, VertexBuffer::AttributeType::FLOAT2, 0, 12)
                .attribute(VertexAttribute::COLOR, 0, VertexBuffer::AttributeType::UBYTE4, 8, 12)
                .normalized(VertexAttribute::COLOR)
                //设置vertexBuffer属性后，通过engine创建vertexBuffer对象
                .build(*engine);
        //填充顶点数据到vertexBuffer对象中(engine对象,vertexBuffer索引，vertexBuffer数据传递函数(顶点数据地址，数据长度3*12字节))        
        app.vb->setBufferAt(*engine, 0,
                VertexBuffer::BufferDescriptor(TRIANGLE_VERTICES, 36, nullptr));
        //设置vertexIndexBuffer顶点索引缓存对象属性  索引数量 数据类型
        app.ib = IndexBuffer::Builder()
                .indexCount(3)
                .bufferType(IndexBuffer::IndexType::USHORT)
                //通过engine对象创建vertexIndexBuffer对象
                .build(*engine);
        //填充索引数据到vertexIndexBuffer对象中(engine对象,vertexIndexBuffer数据传递函数(索引数组，数据长度))
        app.ib->setBuffer(*engine,
                IndexBuffer::BufferDescriptor(TRIANGLE_INDICES, 6, nullptr));
        //创建材质对象
        app.mat = Material::Builder()
                .package(RESOURCES_BAKEDCOLOR_DATA, RESOURCES_BAKEDCOLOR_SIZE)
                .build(*engine);
        //创建renderable可渲染对象                
        app.renderable = EntityManager::get().create();
        //设置可渲染对象的属性
        RenderableManager::Builder(1) //应用于构造器的图元数量
                .boundingBox({{ -1, -1, -1 }, { 1, 1, 1 }})//包围盒用于处理相交测试和碰撞检测
                .material(0, app.mat->getDefaultInstance())//材质索引及实例
                //图元数据装配（索引，图元类型，vertexBuffer,indexBuffer,偏移量，数量）
                .geometry(0, RenderableManager::PrimitiveType::TRIANGLES, app.vb, app.ib, 0, 3)
                .culling(false)
                .receiveShadows(false)
                .castShadows(false)
                .build(*engine, app.renderable);
        scene->addEntity(app.renderable);//通过场景实体管理  renderable对象
        //设置场景相机
        app.camera = utils::EntityManager::get().create();//相机组件
        app.cam = engine->createCamera(app.camera);//相机对象
        view->setCamera(app.cam);
    };

    //析构回收
    auto cleanup = [&app](Engine* engine, View*, Scene*) {
        engine->destroy(app.skybox);
        engine->destroy(app.renderable);
        engine->destroy(app.mat);
        engine->destroy(app.vb);
        engine->destroy(app.ib);
        engine->destroyCameraComponent(app.camera);
        utils::EntityManager::get().destroy(app.camera);
    };

    //更新场景动画
    FilamentApp::get().animate([&app](Engine* engine, View* view, double now) {
        constexpr float ZOOM = 1.5f;
        //设置视口参数
        const uint32_t w = view->getViewport().width;
        const uint32_t h = view->getViewport().height;
        const float aspect = (float) w / h;
        //设置裁剪范围（投影方式，左，右，底，顶，近，远）
        app.cam->setProjection(Camera::Projection::ORTHO,
            -aspect * ZOOM, aspect * ZOOM,
            -ZOOM, ZOOM, 0, 1);
        auto& tcm = engine->getTransformManager();
        //设置变换
        tcm.setTransform(tcm.getInstance(app.renderable),
              filament::math::mat4f::rotation(now, filament::math::float3{ 0, 0, 1 }));
    });
    //主入口
    FilamentApp::get().run(config, setup, cleanup);

    return 0;
}

```    



