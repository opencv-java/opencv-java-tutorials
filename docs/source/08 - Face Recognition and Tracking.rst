=============================
Face Recognition and Tracking
=============================

.. note:: We assume that by now you have already read the previous tutorials. If not, please check previous tutorials at `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_. You can also find the source code and resources at `<https://github.com/opencv-java/>`_

Goal
----
In this tutorial we are going to use well-known classifiers that have been already trained and distributed by OpenCV in order to detect and track a moving face into a video stream.

Cascade Classifiers
-------------------
The object recognition process (in our case, faces) is usually efficient if it is based on the features take-over which include additional information about the object class to be taken-over. In this tutorial we are going to use the *Haar-like features* and the *Local Binary Patterns* (LBP) in order to encode the contrasts highlighted by the human face and its spatial relations with the other objects present in the picture.
Usually these features are extracted using a *Cascade Classifier* which has to be trained in order to recognize with precision different objects: the faces' classification is going to be much different from the car's classification.

What we will do in this tutorial
--------------------------------
In this guide, we will:
 * Insert a checkbox to select the Haar Classifier, detect and track a face, and draw a green rectangle around the detected face.
 * Inesrt a checkbox to select the LBP Classifier, detect and track a face, and draw a green rectangle around the detected face.

Getting Started
---------------
Let's create a new JavaFX project. In Scene Builder set the windows element so that we have a Border Pane with:

- on TOP a VBox a HBox and a separator. In the HBox we are goning to need two checkboxes, the first one is to select the Haar Classifier and the second one is to select the LBP Classifier.

	.. code-block:: xml

		<CheckBox fx:id="haarClassifier" onAction="#haarSelected" text="Haar Classifier"/>
		<CheckBox fx:id="lbpClassifier" onAction="#lbpSelected" text="LBP Classifier"/>

- in the CENTRE we are going to put an ImageView for the web cam stream.

	.. code-block:: xml

		<ImageView fx:id="originalFrame" />

- on the BOTTOM we can add the usual button to start/stop the stream

	.. code-block:: xml

		<Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" disable="true" />

The gui will look something like this one:

.. image:: _static/08-00.png

Loading the Classifiers
-----------------------
First of all we need to add a folder ``resource`` to our project and put the classifiers in it.
In order to use the classifiers we need to load them from the resource folder, so every time that we check one of the two checkboxes we will load the correct classifier.
To do so, let's implement the ``OnAction`` methods we already declared before:

- ``haarSelected``
	inside this method we are going to load the desired Haar Classifier (e.g. ``haarcascade_frontalface.xml``) as follows:

	.. code-block:: java

		this.checkboxSelection("resources/lbpcascades/lbpcascade_frontalface_alt.xml");
		...
		private void checkboxSelection(String... classifierPath)
		{
			// load the classifier(s)
			for (String xmlClassifier : classifierPath)
			{
				this.faceCascade.load(xmlClassifier);
			}

			// now the capture can start
			this.cameraButton.setDisable(false);
		}

- ``lbpSelected``
	for the LPB we can use the same method and change the path of the classifier to be loaded:

	.. code-block:: java

		this.checkboxSelection("resources/lbpcascades/lbpcascade_frontalface.xml");

Detection and Tracking
----------------------
Once we've loaded the classifiers we are ready to start the detection; we are going to implement the detection in the ``detectAndDisplay`` method.
First of all we need to convert the frame in grayscale and equalize the histogram to improve the results:

.. code-block:: java

    Imgproc.cvtColor(frame, grayFrame, Imgproc.COLOR_BGR2GRAY);
    Imgproc.equalizeHist(grayFrame, grayFrame);

Then we have to set the minimum size of the face to be detected (this required is need in the actual detection function). Let's set the minimum size as the 20% of the frame hieght:

.. code-block:: java

    if (this.absoluteFaceSize == 0)
    {
	int height = grayFrame.rows();
	if (Math.round(height * 0.2f) > 0)
	{
		this.absoluteFaceSize = Math.round(height * 0.2f);
	}
    }

Now we can start the detection:

.. code-block:: java

    this.faceCascade.detectMultiScale(grayFrame, faces, 1.1, 2, 0 | Objdetect.CASCADE_SCALE_IMAGE, new Size(this.absoluteFaceSize, this.absoluteFaceSize), new Size());

The ``detectMultiScale`` function detects objects of different sizes in the input image. The detected objects are returned as a list of rectangles.
The parameters are:

 - **image** Matrix of the type CV_8U containing an image where objects are detected.
 - **objects** Vector of rectangles where each rectangle contains the detected object.
 - **scaleFactor** Parameter specifying how much the image size is reduced at each image scale.
 - **minNeighbors** Parameter specifying how many neighbors each candidate rectangle should have to retain it.
 - **flags** Parameter with the same meaning for an old cascade as in the function cvHaarDetectObjects. It is not used for a new cascade.
 - **minSize** Minimum possible object size. Objects smaller than that are ignored.
 - **maxSize** Maximum possible object size. Objects larger than that are ignored.

So the result of the detection is going to be in the **objects** parameter or in our case ``faces``.

Let's put this result in an array of rects and draw them on the frame, by doing so we can display the detected face are:

.. code-block:: java

    Rect[] facesArray = faces.toArray();
    for (int i = 0; i < facesArray.length; i++)
	Core.rectangle(frame, facesArray[i].tl(), facesArray[i].br(), new Scalar(0, 255, 0, 255), 3);

As you can see we selected the color green with a trasparent background: ``Scalar(0, 255, 0, 255)``.
``.tl()`` and ``.br()`` stand for *top-left* and *bottom-right* and they represents the two opposite vertexes.
The last parameter just set the thickness of the rectangle's border.

The tracking part can be implemented by calling the ``detectAndDisplay`` method for each frame.

.. image:: _static/08-01.png

.. image:: _static/08-02.png

Source Code
-----------
-  `FaceDetection.java <https://github.com/opencv-java/face-detection/blob/master/src/it/polito/teaching/cv/Lab5.java>`_

.. code-block:: java

    public class FaceDetection extends Application {
	@Override
	public void start(Stage primaryStage)
	{
		try
		{
			// load the FXML resource
			FXMLLoader loader = new FXMLLoader(getClass().getResource("FD_FX.fxml"));
			BorderPane root = (BorderPane) loader.load();
			// set a whitesmoke background
			root.setStyle("-fx-background-color: whitesmoke;");
			// create and style a scene
			Scene scene = new Scene(root, 800, 600);
			scene.getStylesheets().add(getClass().getResource("application.css").toExternalForm());
			// create the stage with the given title and the previously created
			// scene
			primaryStage.setTitle("Face Detection");
			primaryStage.setScene(scene);
			// show the GUI
			primaryStage.show();

			// init the controller
			FD_Controller controller = loader.getController();
			controller.init();
		}
		catch (Exception e)
		{
			e.printStackTrace();
		}
	}

	public static void main(String[] args)
	{
		// load the native OpenCV library
		System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

		launch(args);
	}
    }

- `FD_Controller.java <https://github.com/opencv-java/face-detection/blob/master/src/it/polito/teaching/cv/FaceDetectionController.java>`_

.. code-block:: java

    public class FD_Controller {
	// FXML buttons
	@FXML
	private Button cameraButton;
	// the FXML area for showing the current frame
	@FXML
	private ImageView originalFrame;
	// checkbox for selecting the Haar Classifier
	@FXML
	private CheckBox haarClassifier;
	// checkbox for selecting the LBP Classifier
	@FXML
	private CheckBox lbpClassifier;

	// a timer for acquiring the video stream
	private Timer timer;
	// the OpenCV object that performs the video capture
	private VideoCapture capture;
	// a flag to change the button behavior
	private boolean cameraActive;
	// the face cascade classifier object
	private CascadeClassifier faceCascade;
	// minimum face size
	private int absoluteFaceSize;
	private Image CamStream;

	/**
	 * Init the controller variables
	 */
	protected void init()
	{
		this.capture = new VideoCapture();
		this.faceCascade = new CascadeClassifier();
		this.absoluteFaceSize = 0;
	}

	/**
	 * The action triggered by pushing the button on the GUI
	 */
	@FXML
	protected void startCamera()
	{
		if (!this.cameraActive)
		{
			// disable setting checkboxes
			this.haarClassifier.setDisable(true);
			this.lbpClassifier.setDisable(true);

			// start the video capture
			this.capture.open(0);

			// is the video stream available?
			if (this.capture.isOpened())
			{
				this.cameraActive = true;

				// grab a frame every 33 ms (30 frames/sec)
				TimerTask frameGrabber = new TimerTask() {
					@Override
					public void run()
					{
						CamStream = grabFrame();
						Platform.runLater(new Runnable() {
							@Override
				            public void run() {

								// show the original frames
								originalFrame.setImage(CamStream);
								// set fixed width
								originalFrame.setFitWidth(600);
								// preserve image ratio
								originalFrame.setPreserveRatio(true);

				            	}
							});
					}
				};
				this.timer = new Timer();
				this.timer.schedule(frameGrabber, 0, 33);

				// update the button content
				this.cameraButton.setText("Stop Camera");
			}
			else
			{
				// log the error
				System.err.println("Failed to open the camera connection...");
			}
		}
		else
		{
			// the camera is not active at this point
			this.cameraActive = false;
			// update again the button content
			this.cameraButton.setText("Start Camera");
			// enable setting checkboxes
			this.haarClassifier.setDisable(false);
			this.lbpClassifier.setDisable(false);

			// stop the timer
			if (this.timer != null)
			{
				this.timer.cancel();
				this.timer = null;
			}
			// release the camera
			this.capture.release();
			// clean the image area
			originalFrame.setImage(null);
		}
	}

	/**
	 * Get a frame from the opened video stream (if any)
	 *
	 * @return the {@link Image} to show
	 */
	private Image grabFrame()
	{
		// init everything
		Image imageToShow = null;
		Mat frame = new Mat();

		// check if the capture is open
		if (this.capture.isOpened())
		{
			try
			{
				// read the current frame
				this.capture.read(frame);

				// if the frame is not empty, process it
				if (!frame.empty())
				{
					// face detection
					this.detectAndDisplay(frame);

					// convert the Mat object (OpenCV) to Image (JavaFX)
					imageToShow = mat2Image(frame);
				}

			}
			catch (Exception e)
			{
				// log the (full) error
				System.err.print("ERROR");
				e.printStackTrace();
			}
		}

		return imageToShow;
	}

	/**
	 * Perform face detection and show a rectangle around the detected face.
	 *
	 * @param frame
	 *            the current frame
	 */
	private void detectAndDisplay(Mat frame)
	{
		// init
		MatOfRect faces = new MatOfRect();
		Mat grayFrame = new Mat();

		// convert the frame in gray scale
		Imgproc.cvtColor(frame, grayFrame, Imgproc.COLOR_BGR2GRAY);
		// equalize the frame histogram to improve the result
		Imgproc.equalizeHist(grayFrame, grayFrame);

		// compute minimum face size (20% of the frame height)
		if (this.absoluteFaceSize == 0)
		{
			int height = grayFrame.rows();
			if (Math.round(height * 0.2f) > 0)
			{
				this.absoluteFaceSize = Math.round(height * 0.2f);
			}
		}

		// detect faces
		this.faceCascade.detectMultiScale(grayFrame, faces, 1.1, 2, 0 | Objdetect.CASCADE_SCALE_IMAGE, new Size(
				this.absoluteFaceSize, this.absoluteFaceSize), new Size());

		// each rectangle in faces is a face
		Rect[] facesArray = faces.toArray();
		for (int i = 0; i < facesArray.length; i++)
			Core.rectangle(frame, facesArray[i].tl(), facesArray[i].br(), new Scalar(0, 255, 0, 255), 3);

	}

	/**
	 * When the Haar checkbox is selected, deselect the other one and load the
	 * proper XML classifier
	 *
	 */
	@FXML
	protected void haarSelected()
	{
		// check whether the lpb checkbox is selected and deselect it
		if (this.lbpClassifier.isSelected())
			this.lbpClassifier.setSelected(false);

		this.checkboxSelection("resources/haarcascades/haarcascade_frontalface_alt.xml");
	}

	/**
	 *
	 When the LBP checkbox is selected, deselect the other one and load the
	 * proper XML classifier
	 */
	@FXML
	protected void lbpSelected()
	{
		// check whether the haar checkbox is selected and deselect it
		if (this.haarClassifier.isSelected())
			this.haarClassifier.setSelected(false);

		this.checkboxSelection("resources/lbpcascades/lbpcascade_frontalface.xml");
	}

	/**
	 * Common operation for both checkbox selections
	 *
	 * @param classifierPath
	 *            the absolute path where the XML file representing a training
	 *            set for a classifier is present
	 */
	private void checkboxSelection(String... classifierPath)
	{
		// load the classifier(s)
		for (String xmlClassifier : classifierPath)
		{
			this.faceCascade.load(xmlClassifier);
		}

		// now the capture can start
		this.cameraButton.setDisable(false);
	}

	/**
	 * Convert a Mat object (OpenCV) in the corresponding Image for JavaFX
	 *
	 * @param frame
	 *            the {@link Mat} representing the current frame
	 * @return the {@link Image} to show
	 */
	private Image mat2Image(Mat frame)
	{
		// create a temporary buffer
		MatOfByte buffer = new MatOfByte();
		// encode the frame in the buffer, according to the PNG format
		Highgui.imencode(".png", frame, buffer);
		// build and return an Image created from the image encoded in the
		// buffer
		return new Image(new ByteArrayInputStream(buffer.toArray()));
	}
    }

- `FD_FX.fxml <https://github.com/opencv-java/face-detection/blob/master/src/it/polito/teaching/cv/FaceDetection.fxml>`_

.. code-block:: xml


    <BorderPane xmlns:fx="http://javafx.com/fxml/1" fx:controller="application.FD_Controller">
	<top>
		<VBox>
			<HBox alignment="CENTER" spacing="10">
				<padding>
					<Insets top="10" bottom="10" />
				</padding>
				<CheckBox fx:id="haarClassifier" onAction="#haarSelected" text="Haar Classifier"/>
				<CheckBox fx:id="lbpClassifier" onAction="#lbpSelected" text="LBP Classifier"/>
			</HBox>
			<Separator />
		</VBox>
	</top>
	<center>
		<VBox alignment="CENTER">
			<padding>
				<Insets right="10" left="10" />
			</padding>
			<ImageView fx:id="originalFrame" />
		</VBox>
	</center>
	<bottom>
		<HBox alignment="CENTER">
			<padding>
				<Insets top="25" right="25" bottom="25" left="25" />
			</padding>
			<Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" disable="true" />
		</HBox>
	</bottom>
    </BorderPane>
