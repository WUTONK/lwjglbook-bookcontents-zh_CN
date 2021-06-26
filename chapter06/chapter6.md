# 变换

## 项目制作

让我们回到上一章中创建的漂亮的彩色四边形, 如果仔细观察, 你会看到四边形是扭曲的, 并显示为矩形. 你甚至可以把窗口的宽度从600像素改为900像素, 这样失真会更明显,这里发生了什么?

如果你重新查看我们的顶点着色器代码, 你会发现我们只是直接传递坐标. 也就是说, 当我们说一个顶点的坐标x值为0.5时, 我们是在说让OpenGL在屏幕上的x位置0.5处绘制它. 下图显示了OpenGL坐标 \(只针对x和y轴\):

![Coordinates](./coordinates.png)

Those coordinates are mapped, considering our window size, to window coordinates \(which have the origin at the top-left corner of the previous figure\). So, if our window has a size of 900x580, OpenGL coordinates \(1,0\) will be mapped to coordinates \(900, 0\) creating a rectangle instead of a quad.
考虑到我们的窗口大小, 这些坐标被映射到窗口坐标\(其原点位于上图的左上角\). 因此,如果我们的窗口大小为900x580,OpenGL坐标\(1,0\)将映射到坐标\(900,0\), 从而创建一个矩形而不是四边形.

![Rectangle](./rectangle.png)

But, the problem is more serious than that. Modify the z coordinate of our quad from 0.0 to 1.0 and to -1.0. What do you see? The quad is exactly drawn in the same place no matter if it’s displaced along the z axis. Why is this happening? Objects that are further away should be drawn smaller than objects that are closer. But we are drawing them with the same x and y coordinates.
但是,问题比这要更严重.尝试将四边形的z坐标从0.0修改为1.0和-1.0.你看到了什么?无论四边形是否沿z轴移,四边形都完全绘制在同一位置.为什么会这样？距离较远的对象应绘制得比距离较近的对象小,但是我们现在是在用相同的x和y坐标来画它们.

But, wait. Should this not be handled by the z coordinate? The answer is yes and no. The z coordinate tells OpenGL that an object is closer or farther away, but OpenGL does not know anything about the size of your object. You could have two objects of different sizes, one closer and smaller and one bigger and further that could be projected correctly onto the screen with the same size \(those would have same x and y coordinates but different z\). OpenGL just uses the coordinates we are passing, so we must take care of this. We need to correctly project our coordinates.
等等,这不应该由z坐标来处理吗？答案是也不是.z坐标告诉OpenGL一个物体是近的还是远的,但是OpenGL对你的物体的大小却一无所知.你可以有两个不同大小的物体,一个更近,一个更小,一个更大,一个更远,它们可以以相同的大小正确地投射到屏幕上\(这些物体的x和y坐标相同,但z坐标不同\).OpenGL只使用我们传递的坐标,所以我们必须注意这个.我们需要正确地投射坐标.

Now that we have diagnosed the problem, how do we fix it? The answer is using a projection matrix or frustum. The projection matrix will take care of the aspect ratio \(the relation between size and height\) of our drawing area so objects won’t be distorted. It also will handle the distance so objects far away from us will be drawn smaller. The projection matrix will also consider our field of view and the maximum distance to be displayed.
既然我们已经诊断出这个问题,我们该如何解决它?答案是使用投影矩阵或截锥.投影矩阵将考虑绘图区域的纵横比\(大小和高度之间的关系\),这样对象就不会失真.它还将处理距离,使远离我们的物体将被画得更小.投影矩阵还将考虑我们的视野和要显示的最大距离.

For those not familiar with matrices, a matrix is a bi-dimensional array of numbers arranged in columns and rows. Each number inside a matrix is called an element. A matrix order is the number of rows and columns. For instance, here you can see a 2x2 matrix \(2 rows and 2 columns\).
对于那些不熟悉矩阵的人来说,矩阵是一个列和行排列成的二维数字数组.矩阵中的每个数字被称为元素,矩阵顺序是行和列的数量.例如,这里可以看到一个2x2矩阵\(2行2列\)

![2x2 Matrix](./2_2_matrix.png)

Matrices have a number of basic operations that can be applied to them \(such as addition, multiplication, etc.\) that you can consult in a math book. The main characteristics of matrices, related to 3D graphics, is that they are very useful to transform points in the space.
矩阵有许多基本运算可以应用于它们\(如加法、乘法等\),你可以在数学书中查阅.矩阵的主要特点与三维图形有关,是它们非常有用的变换空间中的点.

You can think about the projection matrix as a camera, which has a field of view and a minimum and maximum distance. The vision area of that camera will be obtained from a truncated pyramid. The following picture shows a top view of that area.
你可以把投影矩阵想象成一个摄像机,它有一个视野.有一个最小和最大的距离.通过这个“摄像机”把三维的点投射到二维空间上.摄像机的视觉区域将从截短的金字塔中获得.下图显示了该区域的顶视图.

![Projection Matrix concepts](./projection_matrix.png)

A projection matrix will correctly map 3D coordinates so they can be correctly represented on a 2D screen. The mathematical representation of that matrix is as follows \(don’t be scared\).
投影矩阵将正确映射三维坐标,以便在二维屏幕上正确表示.该矩阵的数学表示如下\(不要害怕\).

![Projection Matrix](./projection_matrix_eq.png)

Where aspect ratio is the relation between our screen width and our screen height \($$a=width/height$$\). In order to obtain the projected coordinates of a given point we just need to multiply the projection matrix by the original coordinates. The result will be another vector that will contain the projected version.
其中,纵横比是屏幕宽度和屏幕高度之间的关系,比如说电影屏幕要比一般的屏幕长,那么宽度和高度的比就会相应的变大一点.\($$a=宽度/高度$$\).为了得到给定点的投影坐标,我们只需要将投影矩阵乘以原始坐标.结果将是另一个包含投影版本的向量.

So we need to handle a set of mathematical entities such as vectors, matrices and include the operations that can be done on them. We could choose to write all that code by our own from scratch or use an already existing library. We will choose the easy path and use a specific library for dealing with math operations in LWJGL which is called JOML \(Java OpenGL Math Library\). In order to use that library we just need to add another dependency to our `pom.xml` file.
因此,我们需要处理一组数学实体,比如向量、矩阵,并包含可以对它们执行的操作.我们可以选择自己从头开始编写所有代码,或者使用现有的库.我们将选择简单的路径,并使用一个特定的库来处理LWJGL中的数学操作,这个库称为JOML\(javaopengl数学库\).为了使用这个库,我们只需要在`pom.xml`文件中添加另一个依赖项就好.

```xml
        <dependency>
            <groupId>org.joml</groupId>
            <artifactId>joml</artifactId>
            <version>${joml.version}</version>
        </dependency>
```

And define the version of the library to use.
并定义要使用的库的版本.

```xml
    <properties>
        [...]
        <joml.version>1.10.0</joml.version>
        [...]
    </properties>
```

Now that everything has been set up let’s define our projection matrix. We will create an instance of the class `Matrix4f` \(provided by the JOML library\) in our `Renderer` class. The `Matrix4f` class provides a method to set up a projection matrix named `setPerspective`. This method needs the following parameters:
现在一切都设置好了,让我们定义投影矩阵.我们将在`Renderer`类中创建类`Matrix4f`的实例\(由JOML库提供\).`Matrix4f`类提供了一个方法来设置名为`setPerspective`的投影矩阵.此方法需要以下参数：

* Field of View: The Field of View angle in radians. We will define a constant that holds that value
* 视场：以弧度表示的视场角.我们将定义一个保持该值的常数
* Aspect Ratio.
* 纵横比(画面长宽比).
* Distance to the near plane \(z-near\)
* 到近平面的距离 \(z-近\)
* Distance to the far plane \(z-far\).
* 到远平面的距离\(z-远\).

We will instantiate that matrix in our `init` method so we need to pass a reference to our `Window`  instance to get its size \(you can see it in the source code\). The new constants and variables are:
我们将在`init`方法中实例化该矩阵,因此需要传递对`Window`  实例的引用以获取其大小\(您可以在源代码中看到\).新的常量和变量是：

```java
    /**
     * Field of View in Radians
     */
    private static final float FOV = (float) Math.toRadians(60.0f);

    private static final float Z_NEAR = 0.01f;

    private static final float Z_FAR = 1000.f;

    private Matrix4f projectionMatrix;
```

The projection matrix is created as follows:
投影矩阵创建如下:

```java
float aspectRatio = (float) window.getWidth() / window.getHeight();
projectionMatrix = new Matrix4f().setPerspective(FOV, aspectRatio,
    Z_NEAR, Z_FAR);
```

At this moment we will ignore that the aspect ratio can change \(by resizing our window\). This could be checked in the `render` method to change our projection matrix accordingly.
此时我们将忽略纵横比可以改变\(通过调整窗口大小\).这可以在`render`方法中进行检查,从而相应地更改投影矩阵.

Now that we have our matrix, how do we use it? We need to use it in our shader, and it should be applied to all the vertices. At first, you could think of bundling it in the vertex input \(like the coordinates and the colours\). In this case we would be wasting lots of space since the projection matrix should not change even between several render calls. You may also think of multiplying the vertices by the matrix in the java code. But then, our VBOs would be useless and we will not be using the process power available in the graphics card.
既然我们有了矩阵,我们该如何使用它？我们需要在着色器中使用它,它应该应用于所有顶点.首先,你可以考虑将其绑定到顶点输入\(如坐标和颜色\)中.在这种情况下,我们将浪费大量的空间,因为投影矩阵即使在几个渲染调用之间,也不应该改变.在java代码中,还可以考虑将顶点乘以矩阵.但是那样的话,我们的VBOs将是无用的.我们将不会使用图形卡中可用的处理能力.

The answer is to use “uniforms”. Uniforms are global GLSL variables that shaders can use and that we will employ to communicate with them.
答案是使用“uniforms”.“uniforms”是全局GLSL变量,着色器可以使用它,我们将使用它与它们通信.

So we need to modify our vertex shader code and declare a new uniform called `projectionMatrix` and use it to calculate the projected position.
因此,我们需要修改顶点着色器的代码,并声明一个名为`projectionMatrix`的新统一体,并使用它来计算投影位置.

```glsl
#version 330

layout (location=0) in vec3 position;
layout (location=1) in vec3 inColour;

out vec3 exColour;

uniform mat4 projectionMatrix;

void main()
{
    gl_Position = projectionMatrix * vec4(position, 1.0);
    exColour = inColour;
}
```

As you can see we define our `projectionMatrix` as a 4x4 matrix and the position is obtained by multiplying it by our original coordinates. Now we need to pass the values of the projection matrix to our shader. First, we need to get a reference to the place where the uniform will hold its values.
如你所见,我们将`projectionMatrix`定义为4x4矩阵,位置是通过将其乘以原始坐标获得的.现在我们需要将投影矩阵的值传递给着色器.首先,我们需要得到一个关于统一形式将保存其值的位置的引用.

This is done with the method `glGetUniformLocation` which receives two parameters:
这是通过`glGetUniformLocation`方法完成的,该方法接收两个参数:

* The shader program identifier.
* 着色器程序标识符.
* The name of the uniform \(it should match the one defined in the shader code\).
* 统一形式的名字\(它应该与着色器代码中定义的匹配\).

This method returns an identifier holding the uniform location. Since we may have more than one uniform, we will store those locations in a Map indexed by the location's name \(We will need that location number later\). So in the `ShaderProgram` class we create a new variable that holds those identifiers:
此方法返回保存统一位置的标识符.由于我们可能有不止一个统一的.我们将存储在地图索引的位置的名称\(我们稍后需要那个地址号码\).因此,在`ShaderProgram`类中,我们创建了一个新变量来保存这些标识符：

```java
private final Map<String, Integer> uniforms;
```

This variable will be initialized in our constructor:
此变量将在我们的构造函数中初始化:

```java
uniforms = new HashMap<>();
```

And finally we create a method to set up new uniforms and store the obtained location.
最后,我们建立了一个统一形式的建立方法,并将获得的位置存储起来.

```java
public void createUniform(String uniformName) throws Exception {
    int uniformLocation = glGetUniformLocation(programId,
        uniformName);
    if (uniformLocation < 0) {
        throw new Exception("Could not find uniform:" +
            uniformName);
    }
    uniforms.put(uniformName, uniformLocation);
}
```

Now, in our `Renderer` class we can invoke the `createUniform` method once the shader program has been compiled \(in this case, we will do it once the projection matrix has been instantiated\).
现在,在`Renderer`类中,我们可以在编译着色器程序后调用`createUniform`方法\(在本例中,我们将在投影矩阵实例化之后执行此操作\).

```java
shaderProgram.createUniform("projectionMatrix");
```

At this moment, we already have a holder ready to be set up with data to be used as our projection matrix. Since the projection matrix won’t change between rendering calls we may set up the values right after the creation of the uniform. But we will do it in our render method. You will see later that we may reuse that uniform to do additional operations that need to be done in each render call.
此时此刻,我们已经有了一个支架,准备好用数据作为投影矩阵.由于投影矩阵在渲染调用之间不会改变,因此我们可以在创建统一后立即设置值.但我们将在我们的渲染方法中完成它.稍后,你将看到,我们可以重用该统一来执行需要在每个渲染调用中执行的其他操作.

We will create another method in our `ShaderProgram` class to setup the data, named `setUniform`. Basically we transform our matrix into a 4x4 `FloatBuffer` by using the utility methods provided by the JOML library and send them to the location we stored in our locations map.
我们将在`ShaderProgram`类中创建另一个方法来设置数据,名为`setUniform`.基本上,我们使用JOML库提供的实用方法将矩阵转换为4x4的`FloatBuffer`,并将它们发送到我们存储在位置图中的位置.

```java
public void setUniform(String uniformName, Matrix4f value) {
    // Dump the matrix into a float buffer
    try (MemoryStack stack = MemoryStack.stackPush()) {
        glUniformMatrix4fv(uniforms.get(uniformName), false,
                           value.get(stack.mallocFloat(16)));
    }
}
```

As you can see we are creating buffers in a different way here. We are using auto-managed buffers, and allocating them on the stack. This is due to the fact that the size of this buffer is small and that it will not be used beyond this method. Thus, we use the `MemoryStack` class.
如你所见,我们在这里以不同的方式创建缓冲区.我们使用自动管理的缓冲区.并在堆栈上分配它们.这是因为这个缓冲区的大小很小,并且在这个方法之外不会使用它.因此,我们使用`MemoryStack`类.

Now we can use that method in the `Renderer` class in the `render` method, after the shader program has been bound:
现在我们可以在`render`方法的`Renderer`类中使用该方法,在绑定着色器程序之后:

```java
shaderProgram.setUniform("projectionMatrix", projectionMatrix);
```

We are almost done. We can now show the quad correctly rendered. So you can now launch your program and will obtain a... black background without any coloured quad. What’s happening? Did we break something? Well, actually no. Remember that we are now simulating the effect of a camera looking at our scene. And we provided two distances, one to the farthest plane \(equal to 1000f\) and one to the closest plane \(equal to 0.01f\). Our coordinates are:
我们差不多完成了.现在可以正确地显示四边形.所以你现在可以启动你的程序,并获得一个....黑色背景,一个没有任何颜色的四边形.发生什么事了?我们弄坏什么了吗?嗯..事实上不是.记住,我们现在模拟的是一个摄像头在观察我们的场景的效果.我们提供了两个距离,一个到最远的平面\(等于1000f\),一个到最近的平面\(等于0.01f\).而我们的坐标是：

```java
float[] positions = new float[]{
    -0.5f,  0.5f, 0.0f,
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.5f,  0.5f, 0.0f,
};
```

That is, our z coordinates are outside the visible zone. Let’s assign them a value of `-0.05f`. Now you will see a giant green square like this:
也就是说,我们的z坐标在可见区域之外.我们给它们赋值为`-0.05f`.现在你会看到这样一个巨大的绿色正方形:

![Square 1](./square_1.png)

What is happening now is that we are drawing the quad too close to our camera. We are actually zooming into it. If we assign now a value of `-1.05f` to the z coordinate we can now see our coloured quad.
现在发生的事情是,我们把四边形画得离相机太近了.实际上我们正在放大它.如果我们现在给z坐标赋值为`-1.05f`,我们现在可以看到这个彩色的四边形.

![Square coloured](./square_coloured.png)

If we continue pushing the quad backwards we will see it becoming smaller. Notice also that our quad does not appear as a rectangle anymore.
如果我们继续把相机距离向后推,我们会看到它变小.请注意,我们的四边形不再显示为矩形.

## Applying Transformations

Let’s recall what we’ve done so far. We have learned how to pass data in an efficient format to our graphics card, and how to project that data and assign them colours using vertex and fragments shaders. Now we should start drawing more complex models in our 3D space. But in order to do that we must be able to load an arbitrary model and represent it in our 3D space at a specific position, with the appropriate size and the required rotation.
让我们回忆一下到目前为止我们做了什么.我们已经学习了如何以有效的格式将数据传递到GPU,以及如何使用顶点和片段着色器投影数据,并为其指定颜色.现在我们应该开始在三维空间中绘制更复杂的模型.但为了做到这一点,我们必须能够加载一个任意的模型,并在我们的三维空间中的一个特定的位置,以适当的大小和所需的旋转来表示它.

So right now, in order to do that representation we need to provide some basic operations to act upon any model:
所以现在,为了实现这种表示,我们需要提供一些基本的操作来作用于任何模型:

* Translation: Move an object by some amount on any of the three axes.
* 平移:在三个轴中的任意一个轴上移动物体一定程度.
* Rotation: Rotate an object by some amount of degrees around any of the three axes.
* 旋转:围绕三个轴中的任意一个旋转一个物体一定程度.
* Scale: Adjust the size of an object.
* 缩放:调整对象的大小.

![Transformations](./transformations.png)

The operations described above are known as transformations. And you are probably guessing, correctly, that the way we are going to achieve that is by multiplying our coordinates by a set of matrices \(one for translation, one for rotation and one for scaling\). Those three matrices will be combined into a single matrix called world matrix and passed as a uniform to our vertex shader.
上述操作称为转换.你可能猜对了,我们要达到这个目的的方法是用坐标乘以一组矩阵\(一个用于平移,一个用于旋转,一个用于缩放\).这三个矩阵将组合成一个称为"世界矩阵"的矩阵,并作为一个统一的顶点着色器传递.

The reason why it is called world matrix is because we are transforming from model coordinates to world coordinates. When you learn about loading 3D models you will see that those models are defined in their own coordinate systems. They don’t know the size of your 3D space and they need to be placed in it. So when we multiply our coordinates by our matrix what we are doing is transforming from one coordinate system \(the model one\) to another \(the one for our 3D world\).
之所以称之为世界矩阵,是因为我们正在从模型坐标转换为世界(指游戏世界)坐标.当你了解如何加载三维模型时,你将看到这些模型是在它们自己的坐标系中定义的.他们不知道你的三维空间的大小,他们需要放在里面.所以当我们把坐标乘以我们的矩阵时,我们所做的是从一个坐标系\(这些模型\)转换到另一个坐标系\(三维世界的坐标系\).

That world matrix will be calculated like this \(the order is important since multiplication using matrices is not commutative\):
世界矩阵将这样计算\(顺序很重要,因为使用矩阵的乘法不是可交换的\):

注释:matrix为矩阵 Translation为平移 Rotation为旋转 Scal为缩放

$$
World Matrix=\left[Translation Matrix\right]\left[Rotation Matrix\right]\left[Scale Matrix\right]
世界矩阵=\左[平移矩阵\右]\左[旋转矩阵\右]\左[缩放矩阵\右]
$$


If we include our projection matrix in the transformation matrix it would be like this:
如果我们在变换矩阵中包含投影矩阵,它会是这样的:

$$
\begin{array}{lcl}
Transf & = & \left[Proj Matrix\right]\left[Translation Matrix\right]\left[Rotation Matrix\right]\left[Scale Matrix\right] \\
 & = & \left[Proj Matrix\right]\left[World Matrix\right]
\end{array}
$$



The translation matrix is defined like this:
平移矩阵的定义如下:

$$
\begin{bmatrix}
1 & 0 & 0 & dx \\
0 & 1 & 0 & dy \\
0 & 0 & 1 & dz \\
0 & 0 & 0 & 1
\end{bmatrix}
$$


Translation Matrix Parameters:
平移矩阵参数：

* dx: Displacement along the x axis.
* dx: 沿x轴的位移.
* dy: Displacement along the y axis.
* dy: 沿y轴的位移.
* dz: Displacement along the z axis.
* dz: 沿z轴的位移.

The scale matrix is defined like this:


$$
\begin{bmatrix}
sx & 0 & 0 & 0 \\
0 & sy & 0 & 0 \\
0 & 0 & sz & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$


Scale Matrix Parameters:
缩放矩阵的定义如下：

* sx: Scaling along the x axis.
* sx:沿x轴缩放.
* sy: Scaling along the y axis.
* sy:沿y轴缩放.
* sz: Scaling along the z axis.
* sz:沿z轴缩放.

The rotation matrix is much more complex. But keep in mind that it can be constructed by the multiplication of 3 rotation matrices for a single axis, each.
旋转矩阵要复杂得多,但请记住.它可以由一个轴的3个旋转矩阵相乘来构造旋转矩阵(每个).

Now, in order to apply those concepts we need to refactor our code a little bit. In our game we will be loading a set of models which can be used to render many objects at different positions according to our game logic \(imagine a FPS game which loads three models for different enemies. There are only three models but using these models we can draw as many enemies as we want\). Do we need to create a VAO and the set of VBOs for each of those objects? The answer is no. We only need to load it once per model. What we need to do is to draw it independently according to its position, size and rotation. We need to transform those models when we are rendering them.
现在,为了应用这些概念,我们需要稍微重构一下代码.在我们的游戏中,我们将加载一组模型,这些模型可以用来根据我们的游戏逻辑,渲染不同位置的许多对象\(想象一个FPS游戏,我们要为不同的敌人加载三个模型.我们也只有三个模型,但使用这些模型,我们可以画尽可能多的敌人\).我们需要为每个对象创建一个VAO和一组VBO吗?答案是否定的.我们只需要为每个模型加载一次就好.我们需要做的是根据它的位置、大小和旋转来独立地绘制它.我们需要在渲染这些模型时对其进行变换.


So we will create a new class named `GameItem` that will hold a reference to a model, to a `Mesh` instance. A `GameItem` instance will have variables for storing its position, its rotation state and its scale. This is the definition of that class.
因此,我们将创建一个名为`GameItem`的新类,该类将包含对模型和`Mesh`实例的引用.`GameItem`实例将具有用于存储其位置、旋转状态和缩放的变量.这就是那个类的定义.

```java
package org.lwjglb.engine;

import org.joml.Vector3f;
import org.lwjglb.engine.graph.Mesh;

public class GameItem {

    private final Mesh mesh;

    private final Vector3f position;

    private float scale;

    private final Vector3f rotation;

    public GameItem(Mesh mesh) {
        this.mesh = mesh;
        position = new Vector3f();
        scale = 1;
        rotation = new Vector3f();
    }

    public Vector3f getPosition() {
        return position;
    }

    public void setPosition(float x, float y, float z) {
        this.position.x = x;
        this.position.y = y;
        this.position.z = z;
    }

    public float getScale() {
        return scale;
    }

    public void setScale(float scale) {
        this.scale = scale;
    }

    public Vector3f getRotation() {
        return rotation;
    }

    public void setRotation(float x, float y, float z) {
        this.rotation.x = x;
        this.rotation.y = y;
        this.rotation.z = z;
    }

    public Mesh getMesh() {
        return mesh;
    }
}
```

We will create another class which will deal with transformations named `Transformation`.
我们将创建另一个类来处理名为`Transformation`的转换.

```java
package org.lwjglb.engine.graph;

import org.joml.Matrix4f;
import org.joml.Vector3f;

public class Transformation {

    private final Matrix4f projectionMatrix;

    private final Matrix4f worldMatrix;

    public Transformation() {
        worldMatrix = new Matrix4f();
        projectionMatrix = new Matrix4f();
    }

    public final Matrix4f getProjectionMatrix(float fov, float width, float height, float zNear, float zFar) {
        return projectionMatrix.setPerspective(fov, width / height, zNear, zFar);
    }

    public Matrix4f getWorldMatrix(Vector3f offset, Vector3f rotation, float scale) {
        return worldMatrix.translation(offset).
                rotateX((float)Math.toRadians(rotation.x)).
                rotateY((float)Math.toRadians(rotation.y)).
                rotateZ((float)Math.toRadians(rotation.z)).
                scale(scale);
    }
}
```

As you can see this class groups the projection and world matrices. Given a set of vectors that model the displacement, rotation and scale it returns the world matrix. The method `getWorldMatrix` returns the matrix that will be used to transform the coordinates for each `GameItem` instance. That class also provides a method that gets the projection matrix based on the Field Of View, the aspect ratio and the near and far distances.
如你所见,这个类将投影矩阵和世界矩阵分组.给定一组模拟位移、旋转和缩放的向量,它将返回世界矩阵.方法`getWorldMatrix`返回用于转换每个“GameItem”实例坐标的矩阵.该类还提供了一种基于视场、纵横比和远近距离获取投影矩阵的方法.

An important thing to notice is that the `mul` method of the `Matrix4f` class modifies the matrix instance which the method is being applied to. So if we directly multiply the projection matrix with the transformation matrix we will modify the projection matrix itself. This is why we are always initializing each matrix to the identity matrix upon each call.
需要注意的一点是,`Matrix4f`类的`mul`方法修改了应用该方法的矩阵实例.所以如果我们直接把投影矩阵和变换矩阵相乘,我们会修改投影矩阵本身.这就是为什么每次调用时,我们总是将每个矩阵初始化为标识矩阵.

In the `Renderer` class, in the constructor method, we just instantiate the `Transformation` with no arguments and in the `init` method we just create the uniform.
在`Renderer`类的构造函数方法中,我们只是实例化`Transformation`,而在`init`方法中,我们只是创建统一的.

```java
public Renderer() {
    transformation = new Transformation();
}

public void init(Window window) throws Exception {
    // .... Some code before ...
    // Create uniforms for world and projection matrices
    shaderProgram.createUniform("projectionMatrix");
    shaderProgram.createUniform("worldMatrix");

    window.setClearColor(0.0f, 0.0f, 0.0f, 0.0f);
}
```

In the render method of our `Renderer` class we now receive an array of GameItems:
在`Renderer`类的render方法中,我们现在接收一个GameItems数组:

```java
public void render(Window window, GameItem[] gameItems) {
    clear();

    if (window.isResized()) {
        glViewport(0, 0, window.getWidth(), window.getHeight());
        window.setResized(false);
    }

    shaderProgram.bind();

    // Update projection Matrix
    Matrix4f projectionMatrix = transformation.getProjectionMatrix(FOV, window.getWidth(), window.getHeight(), Z_NEAR, Z_FAR);
    shaderProgram.setUniform("projectionMatrix", projectionMatrix);        

    // Render each gameItem
    for (GameItem gameItem : gameItems) {
        // Set world matrix for this item
        Matrix4f worldMatrix =
            transformation.getWorldMatrix(
                gameItem.getPosition(),
                gameItem.getRotation(),
                gameItem.getScale());
        shaderProgram.setUniform("worldMatrix", worldMatrix);
        // Render the mesh for this game item
        gameItem.getMesh().render();
    }

    shaderProgram.unbind();
}
```

We update the projection matrix once per `render` call. By doing it this way we can deal with window resize operations. Then we iterate over the `GameItem` array and create a transformation matrix according to the position, rotation and scale of each of them. This matrix is pushed to the shader and the `Mesh` is drawn. The projection matrix is the same for all the items to be rendered. This is the reason why it’s a separate variable in our `Transformation` class.
每次`render`被调用,就更新一次投影矩阵.通过这样做,我们可以处理窗口大小调整操作.然后我们迭代`GameItem`数组,并根据每个数组的位置、旋转和缩放创建一个变换矩阵.将此矩阵推送到着色器并绘制“网格”.投影矩阵对于要渲染的所有项都是相同的.这就是为什么它在我们的`Transformation`类中是一个单独的变量.

We moved the rendering code to draw a `Mesh` to its class:
我们将渲染代码和`Mesh`移到类中以绘制网格:

```java
public void render() {
    // Draw the mesh
    glBindVertexArray(getVaoId());

    glDrawElements(GL_TRIANGLES, getVertexCount(), GL_UNSIGNED_INT, 0);

    // Restore state
    glBindVertexArray(0);
}
```

Our vertex shader is modified by simply adding a new `worldMatrix` matrix and it uses it with the `projectionMatrix` to calculate the position:
我们的顶点着色器只需添加一个新的`worldMatrix`矩阵即可进行修改,并将其与`projectionMatrix`一起用于计算位置:

```glsl
#version 330

layout (location=0) in vec3 position;
layout (location=1) in vec3 inColour;

out vec3 exColour;

uniform mat4 worldMatrix;
uniform mat4 projectionMatrix;

void main()
{
    gl_Position = projectionMatrix * worldMatrix * vec4(position, 1.0);
    exColour = inColour;
}
```

As you can see the code is exactly the same. We are using the uniform to correctly project our coordinates taking into consideration our frustum, position, scale and rotation information.
正如你所看到的,代码是完全相同的.我们正在使用统一的项目,正确地考虑到我们的截锥,位置,比例和旋转信息和我们的坐标.

Another important thing to think about is, why don’t we pass the translation, rotation and scale matrices instead of combining them into a world matrix? The reason is that we should try to limit the matrices we use in our shaders. Also keep in mind that the matrix multiplication that we do in our shader is done once per each vertex. The projection matrix does not change between render calls and the world matrix does not change per `GameItem` instance. If we passed the translation, rotation and scale matrices independently we would be doing many more matrix multiplications. Think about a model with tons of vertices. That’s a lot of extra operations.
还有一个需要考虑的重要问题是,为什么我们不选择传递平移、旋转和缩放矩阵,而是将它们组合成一个世界矩阵?原因是我们应该尝试限制我们在着色器中使用的矩阵.还请记住,我们在着色器中执行的矩阵乘法会在每个顶点执行一次.投影矩阵不会在渲染调用之间更改,并且世界矩阵不会在每个`GameItem`实例中更改.如果我们独立地传递平移、旋转和缩放矩阵,我们将进行更多的矩阵乘法.考虑一个有大量顶点的模型.那会是很多额外的操作.

But you may think now that if the world matrix does not change per `GameItem` instance, why didn't we do the matrix multiplication in our Java class? We could multiply the projection matrix and the world matrix just once per `GameItem` and send it as a single uniform. In this case we would be saving many more operations, right? The answer is that this is a valid point for now, but when we add more features to our game engine we will need to operate with world coordinates in the shaders anyway, so it’s better to handle those two matrices in an independent way.
但是现在你可能会觉得,如果每个`GameItem`实例的世界矩阵不变,为什么我们不在Java类中执行矩阵乘法呢?我们可以将投影矩阵和世界矩阵相乘一次,然后将其作为一个统一的单位发送.在这种情况下,我们将节省更多的操作,对吗?答案是,这在现在也许奏效,但当我们添加更多的功能到我们的游戏引擎,无论如何,我们都需要在着色器中使用世界坐标,所以最好是以独立的方式处理这两个矩阵.

Finally we only need to change the `DummyGame` class to create an instance of `GameItem` with its associated `Mesh` and add some logic to translate, rotate and scale our quad. Since it’s only a test example and does not add too much you can find it in the source code that accompanies this book.
最后,我们只需要更改`DummyGame`类来创建一个带有关联`Mesh`的`GameItem`实例,并添加一些逻辑来平移、旋转和缩放四边形.因为它只是一个测试示例,没有添加太多内容,所以你可以在本书附带的源代码中找到它.
