=========================================
Your First JavaFX Application with OpenCV
=========================================

.. note:: We assume that by now you have already read the previous tutorials. If not, please check previous tutorials at `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_. You can also find the source code and resources at `<https://github.com/opencv-java/>`_

A JavaFX application with OpenCV
--------------------------------
This tutorial will guide you through the creation of a simple JavaFX GUI application using the  OpenCV library in Eclipse.

What we will do in this tutorial
--------------------------------
In this guide, we will:
 * Install the ``e(fx)clipse`` plugin and (optionally) *Scene Builder*.
 * Work with *Scene Builder*.
 * Write and Run our application.

Your First Application in JavaFX
--------------------------------
The application you will write by following this tutorial is going to capture a video stream from a webcam and, then, it will display it on the user interface (GUI). We will create the GUI with Scene Builder: it will have a button, which will allow us to start and stop the stream, and a simple image view container where we will put each stream frame.

Installing e(fx)clipse plugin and Scene Builder
-----------------------------------------------
In Eclipse, install the ``e(fx)clipse`` plugin, by following the guide at `<http://www.eclipse.org/efxclipse/install.html#fortheambitious>`_.
If you choose not to install such a plugin, you have to create a traditional **Java project**, only.
Download and install *JavaFX Scene Builder* 2.0 from `<http://www.oracle.com/technetwork/java/javafxscenebuilder-1x-archive-2199384.html>`_.

Now you can create a new JavaFX project. ``Go to File > New > Project...`` and select ``JavaFX project...``.

.. image:: _static/03-00.png

Choose a name for your project and click ``Next``.

.. image:: _static/03-01.png

Now add your OpenCV user library to your project and click ``Next``.

.. image:: _static/03-02.png

Choose a name for your package, for the *FXML file* and for the *Controller Class*.
The *FXML file* will contain the description of your GUI in FXML language, while the *Controller Class* will handle all the method and event which have to be called and managed when the user interacts with the GUI's components.

.. image:: _static/03-03.png

Working with Scene Builder
--------------------------
If you have installed *Scene Builder* you can now right click on your *FXML file* in Eclipse and select ``Open with SceneBuilder``.
*Scene Builder* can help construct you gui by interacting with a graphic interface; this allows you to see a real time preview of your window and modify your components and their position just by editing the graphic preview. Let's take a look at what I'm talking about.
At fist the *FXML file* will have just an *AnchorPane*.
An AnchorPane allows the edges of child nodes to be anchored to an offset from the anchorpane's edges. If the anchorpane has a border and/or padding set, the offsets will be measured from the inside edge of those insets.
The anchorpane lays out each managed child regardless of the child's visible property value; unmanaged children are ignored for all layout calculations.
You can go ahead and delete the anchorpane and add a *BorderPane* instead.
A BorderPane lays out children in top, left, right, bottom, and center positions.

.. image:: _static/03-04.png

You can add a BorderPane by dragging from the ``Container`` menu a borderpane and then drop it in the ``Hierarchy`` menu.
Now we can add the button that will allow us to start and stop the stream. Take a button component from the ``Controls`` menu and drop it on the **BOTTOM** field of our BP.
As we can see, on the right we will get three menus (Properties, Layout, Code) which are used to customize our selected component.
For example we can change text of our button in "Start Camera" in the ``Text`` field under the ``Properties`` menu and the id of the button (e.g. "start_btn") in the ``fx:id`` field under the ``Code`` menu.

.. image:: _static/03-05.png

.. image:: _static/03-06.png

We are going to need the id of the button later, in order to edit the button properties from our *Controller*'s methods.
As you can see our button is too close to the edge of the windows, so we should add some bottom margin to it; to do so we can add this information in the ``Layout`` menu.
In order to make the button work, we have to set the name of the method (e.g. "startCamera") that will execute the action we want to preform in the field ``OnAction`` under the ``Code`` menu.

.. image:: _static/03-07.png

Now, we shall add an *ImageView* component from the ``Controls`` menu into the **CENTER** field of our BP. Let's also edit the id of the image view (e.g. "currentFrame"), and add some margin to it.

.. image:: _static/03-08.png

Finally we have to tell which Controller class will mange the GUI, we can do so by adding our controller class name in the ``Controller class`` field under the ``Controller`` menu located in the bottom left corner of the window.

We just created our first GUI by using Scene Builder, if you save the file and return to Eclipse you will notice that some FXML code has been generated automatically.

Key Concepts in JavaFX
----------------------
The **Stage** is where the application will be displayed (e.g., a Windows' window).
A **Scene** is one container of Nodes that compose one "page" of your application.
A **Node** is an element in the Scene, with a visual appearance and an interactive behavior. Nodes may be hierarchically nested .
In  the *Main class* we have to pass to the *start* function our *primary stage*:

.. code-block:: java

    public void start(Stage primaryStage)

and load the fxml file that will populate our stage, the *root element* of the scene and the controller class:

.. code-block:: java

    FXMLLoader loader = new FXMLLoader(getClass().getResource("FXHelloCV.fxml"));
    BorderPane root = (BorderPane) loader.load();
    FXController controller = loader.getController();

Managing GUI Interactions With the Controller Class
---------------------------------------------------
For our application we need to do basically two thing: control the button push and the refreshment of the image view.
To do so we have to create a reference between the gui components and a variable used in our controller class:

.. code-block:: java

    @FXML
    private Button button;
    @FXML
    private ImageView currentFrame;

The ``@FXML`` tag means that we are linking our variable to an element of the fxml file and the value used to declare the variable has to equal to the id set for that specific element.

The ``@FXML`` tag is used with the same meaning for the Actions set under the Code menu in a specific element.

for:

.. code-block:: xml

    <Button fx:id="button" mnemonicParsing="false" onAction="#startCamera" text="Start Camera" BorderPane.alignment="CENTER">

we set:

.. code-block:: java

    @FXML
    protected void startCamera(ActionEvent event) { ...

Video Capturing
---------------
Essentially, all the functionalities required for video manipulation is integrated in the VideoCapture class.

.. code-block:: java

    private VideoCapture capture = new VideoCapture();

This on itself builds on the FFmpeg open source library. A video is composed of a succession of images, we refer to these in the literature as frames. In case of a video file there is a frame rate specifying just how long is between two frames. While for the video cameras usually there is a limit of just how many frames they can digitalize per second.
In our case we set as frame rate 30 frames per sec. To do so we initialize a timer (i.e., a ```ScheduledExecutorService```) that will open a background task every *33 milliseconds*.

.. code-block:: java

    Runnable frameGrabber = new Runnable() { ... }
    this.timer = Executors.newSingleThreadScheduledExecutor();
		this.timer.scheduleAtFixedRate(frameGrabber, 0, 33, TimeUnit.MILLISECONDS);

To check if the binding of the class to a video source was successful or not use the ``isOpened`` function:

.. code-block:: java

    if (this.capture.isOpened()) { ... }

Closing the video is automatic when the objects destructor is called. However, if you want to close it before this you need to call its release function.

.. code-block:: java

    this.capture.release();

The frames of the video are just simple images. Therefore, we just need to extract them from the VideoCapture object and put them inside a Mat one.

.. code-block:: java

    Mat frame = new Mat();

The video streams are sequential. You may get the frames one after another by the read or the overloaded >> operator.

.. code-block:: java

    this.capture.read(frame);

Now we are going to convert our image from *BGR* to *Grayscale* format. OpenCV has a really nice function to do this kind of transformations:

.. code-block:: java

    Imgproc.cvtColor(frame, frame, Imgproc.COLOR_BGR2GRAY);

As you can see, cvtColor takes as arguments:
 - a source image (frame)
 - a destination image (frame), in which we will save the converted image.
 - an additional parameter that indicates what kind of transformation will be performed. In this case we use ``COLOR_BGR2GRAY`` (because of ``imread`` has BGR default channel order in case of color images).

Now in order to put the captured frame into the ImageView we need to convert the Mat in a Image.
We first create a buffer to store the Mat.

.. code-block:: java

    MatOfByte buffer = new MatOfByte();

Then we can put the frame into the buffer by using the ``imencode`` function:

.. code-block:: java

    Imgcodecs.imencode(".png", frame, buffer);

This encodes an image into a memory buffer. The function compresses the image and stores it in the memory buffer that is resized to fit the result.

.. note:: ``imencode`` returns single-row matrix of type ``CV_8UC1`` that contains encoded image as array of bytes.

It takes three parameters:
 - (".png") File extension that defines the output format.
 - (frame) Image to be written.
 - (buffer) Output buffer resized to fit the compressed image.

Once we filled the buffer we have to stream it into an Image by using ``ByteArrayInputStream``:

.. code-block:: java

    new Image(new ByteArrayInputStream(buffer.toArray()));

Now we can put the new image in the ImageView.
With *Java 1.8* we cannot perform an update of a GUI element in a thread that differs from the main thread; so we need to get the new frame in a second thread and refresh our ImageView in the main thread:

.. code-block:: java

    Image imageToShow = grabFrame();
    Platform.runLater(new Runnable() {
	    @Override public void run() { currentFrame.setImage(imageToShow); }
    });

.. image:: _static/03-09.png

The source code of the entire tutorial is available on `GitHub <https://github.com/opencv-java/getting-started/blob/master/FXHelloCV/>`_.
