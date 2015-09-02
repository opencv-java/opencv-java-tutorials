=================
Object Detection
=================

.. note:: We assume that by now you have already read the previous tutorials. If not, please check previous tutorials at `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_. You can also find the source code and resources at `<https://github.com/opencv-java/>`_

Goal
----
In this tutorial we are going to identify and track one or more tennis balls. It performs the detection of the tennis balls upon a webcam video stream by using the color range of the balls, erosion and dilation, and the findContours method.

Morphological Image Processing
------------------------------
Is a collection of non-linear operations related to the morphology of features in an image. The morphological operations rely only on the relative ordering of pixel values and not on their numerical values.
Some of the fundamental morphological operations are dilation and erosion. Dilation causes objects to dilate or grow in size adding pixels to the boundaries of objects in an image and therefore the holes within different regions become smaller. The dilation allows, for example, to join parts of an object that appear separated.
Erosion causes objects to shrink by stripping away layers of pixels from the boundaries of objects in an image and therefore the holes within different regions become larger. The erosion can be used to remove noise or small errors from the image due to the scanning process.
The opening is a compound operation that consist in an erosion followed by a dilation using the same structuring element for both operations. This operation removes small objects from the foreground of an image and can be used to find things into which a specific structuring element can fit. The opening can open up a gap between objects connected by a thin bridge of pixels. Any regions that have survived the erosion are restored to their original size by the dilation.


What we will do in this tutorial
--------------------------------
In this guide, we will:
 * Insert 3 groups of sliders to control the quantity of HSV (Hue, Saturation and Value) of the image.
 * Capture and process the image from the web cam removing noise in order to facilitate the object recognition.
 * Finally using morphological operator such as erosion and dilation we can identify the objects using the contornous obtained after the image processing.

Getting Started
---------------
Let's create a new JavaFX project. In Scene Builder set the windows element so that we have a Border Pane with:

- on RIGHT CENTER we can add a VBox. in this one we are goning to need 6 sliders, the first couple will control hue, the next one saturation and finally hue, with these sliders is posibly change the values of the HSV image.

	.. code-block:: xml

               	<Label text="Hue Start" />
		<Slider fx:id="hueStart" min="0" max="180" value="20" blockIncrement="1" />
		<Label text="Hue Stop" />
		<Slider fx:id="hueStop" min="0" max="180" value="50" blockIncrement="1" />
		<Label text="Saturation Start" />
		<Slider fx:id="saturationStart" min="0" max="255" value="60" blockIncrement="1" />
		<Label text="Saturation Stop" />
		<Slider fx:id="saturationStop" min="0" max="255" value="200" blockIncrement="1" />
		<Label text="Value Start" />
		<Slider fx:id="valueStart" min="0" max="255" value="50" blockIncrement="1" />
		<Label text="Value Stop" />
		<Slider fx:id="valueStop" min="0" max="255" value="255" blockIncrement="1" />	


- in the CENTER. we are going to put tree ImageViews the first one show normal image from the web cam stream, the second one will show mask image and the lastone will show morph image. The HBox is used to normal image and VBox to put the other ones. 

	.. code-block:: xml

		<HBox alignment="CENTER" spacing="5">
			<padding>
				<Insets right="10" left="10" />
			</padding>
			<ImageView fx:id="originalFrame" />
			<VBox alignment="CENTER" spacing="5">
				<ImageView fx:id="maskImage" />
				<ImageView fx:id="morphImage" />
			</VBox>
		</HBox>

- on the BOTTOM we can add the usual button to start/stop the stream and the currents values HSV selected with the sliders.

	.. code-block:: xml

		<Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" />
		<Separator />
		<Label fx:id="hsvCurrentValues" />

The gui will look something like this one:

.. image:: _static/09-00.png


Image processing
----------------
First of all we need to add a folder ``resource`` to our project and put the classifiers in it.
In order to use the classifiers we need to load them from the resource folder, so every time that we check one of the two checkboxes we will load the correct classifier.

- ``Remove noise``
	We can remove some noise of the image using the method blur of the Imgproc class and then apply a conversion to 
	HSV in order to facilitated the process of object recognition.

	.. code-block:: java
	
		Mat blurredImage = new Mat();
		Mat hsvImage = new Mat();
		Mat mask = new Mat();
		Mat morphOutput = new Mat();
					
		// remove some noise
		Imgproc.blur(frame, blurredImage, new Size(7, 7));
					
		// convert the frame to HSV
		Imgproc.cvtColor(blurredImage, hsvImage, Imgproc.COLOR_BGR2HSV);
	
	

- ``Values of HSV image``
	With the sliders we can modify the values of the HSV Image, the image will be updtated in real time,
	that allow increase or decrease the capactity to recognize object into the image. .

	.. code-block:: java
	
	
		// get thresholding values from the UI
		// remember: H ranges 0-180, S and V range 0-255
		Scalar minValues = new Scalar(this.hueStart.getValue(), this.saturationStart.getValue(),
		this.valueStart.getValue());
		Scalar maxValues = new Scalar(this.hueStop.getValue(), this.saturationStop.getValue(),
		this.valueStop.getValue());
				
		// show the current selected HSV range
		String valuesToPrint = "Hue range: " + minValues.val[0] + "-" + maxValues.val[0]
		+ "\tSaturation range: " + minValues.val[1] + "-" + maxValues.val[1] + "\tValue range: "
		+ minValues.val[2] + "-" + maxValues.val[2];
		this.onFXThread(this.hsvValuesProp, valuesToPrint);
				
		// threshold HSV image to select tennis balls
		Core.inRange(hsvImage, minValues, maxValues, mask);
		// show the partial output
		this.onFXThread(maskProp, this.mat2Image(mask));
		

Morphological Operators
-----------------------
First of all we need to add a folder ``resource`` to our project and put the classifiers in it.
In order to use the classifiers we need to load them from the resource folder, so every time that we check one of the two checkboxes we will load the correct classifier.


	.. code-block:: java
		
	       // morphological operators
	       // dilate with large element, erode with small ones
	        Mat dilateElement = Imgproc.getStructuringElement(Imgproc.MORPH_RECT, new Size(24, 24));
		Mat erodeElement = Imgproc.getStructuringElement(Imgproc.MORPH_RECT, new Size(12, 12));
					
		Imgproc.erode(mask, morphOutput, erodeElement);
		Imgproc.erode(mask, morphOutput, erodeElement);
				
		Imgproc.dilate(mask, morphOutput, dilateElement);
		Imgproc.dilate(mask, morphOutput, dilateElement);
				
		// show the partial output
		this.onFXThread(this.morphProp, this.mat2Image(morphOutput));
		
		

Trackin the Object
------------------
Given a binary image containing one or more closed surfaces, use it as a mask to find and highlight the objects contours.


	.. code-block:: java
	
		// init
		List<MatOfPoint> contours = new ArrayList<>();
		Mat hierarchy = new Mat();
		
		// find contours
		Imgproc.findContours(maskedImage, contours, hierarchy, Imgproc.RETR_CCOMP, Imgproc.CHAIN_APPROX_SIMPLE);
		
		// if any contour exist...
		if (hierarchy.size().height > 0 && hierarchy.size().width > 0)
		{
			// for each contour, display it in blue
			for (int idx = 0; idx >= 0; idx = (int) hierarchy.get(0, idx)[0])
			{
				Imgproc.drawContours(frame, contours, idx, new Scalar(250, 0, 0));
			}
		}


Source Code
-----------
-  `ObjectDetection.java <https://github.com/opencv-java/object-detection/blob/master/src/it/polito/teaching/cv/Lab7.java>`_

.. code-block:: java

	    public class ObjectDetection extends Application
	{
	
		@Override
		public void start(Stage primaryStage)
		{
			try
			{
				// load the FXML resource
				BorderPane root = (BorderPane) FXMLLoader.load(getClass().getResource("ObjRecognition.fxml"));
				// set a whitesmoke background
				root.setStyle("-fx-background-color: whitesmoke;");
				// create and style a scene
				Scene scene = new Scene(root, 800, 600);
				scene.getStylesheets().add(getClass().getResource("application.css").toExternalForm());
				// create the stage with the given title and the previously created
				// scene
				primaryStage.setTitle("Object Detection");
				primaryStage.setScene(scene);
				// show the GUI
				primaryStage.show();
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

- `ObjRecognitionController.java <https://github.com/opencv-java/object-detection/blob/master/src/it/polito/teaching/cv/ObjRecognitionController.java>`_

.. code-block:: java

	    public class ObjRecognitionController
	{
		// FXML camera button
		@FXML
		private Button cameraButton;
		// the FXML area for showing the current frame
		@FXML
		private ImageView originalFrame;
		// the FXML area for showing the mask
		@FXML
		private ImageView maskImage;
		// the FXML area for showing the output of the morphological operations
		@FXML
		private ImageView morphImage;
		// FXML slider for setting HSV ranges
		@FXML
		private Slider hueStart;
		@FXML
		private Slider hueStop;
		@FXML
		private Slider saturationStart;
		@FXML
		private Slider saturationStop;
		@FXML
		private Slider valueStart;
		@FXML
		private Slider valueStop;
		// FXML label to show the current values set with the sliders
		@FXML
		private Label hsvCurrentValues;
		
		// a timer for acquiring the video stream
		private Timer timer;
		// the OpenCV object that performs the video capture
		private VideoCapture capture = new VideoCapture();
		// a flag to change the button behavior
		private boolean cameraActive;
		
		// property for object binding
		private ObjectProperty<Image> maskProp;
		private ObjectProperty<Image> morphProp;
		private ObjectProperty<String> hsvValuesProp;
		
		/**
		 * The action triggered by pushing the button on the GUI
		 */
		@FXML
		private void startCamera()
		{
			// bind an image property with the original frame container
			final ObjectProperty<Image> imageProp = new SimpleObjectProperty<>();
			this.originalFrame.imageProperty().bind(imageProp);
			
			// bind an image property with the mask container
			maskProp = new SimpleObjectProperty<>();
			this.maskImage.imageProperty().bind(maskProp);
			
			// bind an image property with the container of the morph operators
			// output
			morphProp = new SimpleObjectProperty<>();
			this.morphImage.imageProperty().bind(morphProp);
			
			// bind a text property with the string containing the current range of
			// HSV values for object detection
			hsvValuesProp = new SimpleObjectProperty<>();
			this.hsvCurrentValues.textProperty().bind(hsvValuesProp);
			
			// set a fixed width for all the image to show and preserve image ratio
			this.imageViewProperties(this.originalFrame, 400);
			this.imageViewProperties(this.maskImage, 200);
			this.imageViewProperties(this.morphImage, 200);
			
			if (!this.cameraActive)
			{
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
							// update the image property => update the frame
							// shown in the UI
							Image frame = grabFrame();
							onFXThread(imageProp, frame);
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
				
				// stop the timer
				if (this.timer != null)
				{
					this.timer.cancel();
					this.timer = null;
				}
				// release the camera
				this.capture.release();
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
						// init
						Mat blurredImage = new Mat();
						Mat hsvImage = new Mat();
						Mat mask = new Mat();
						Mat morphOutput = new Mat();
						
						// remove some noise
						Imgproc.blur(frame, blurredImage, new Size(7, 7));
						
						// convert the frame to HSV
						Imgproc.cvtColor(blurredImage, hsvImage, Imgproc.COLOR_BGR2HSV);
						
						// get thresholding values from the UI
						// remember: H ranges 0-180, S and V range 0-255
						Scalar minValues = new Scalar(this.hueStart.getValue(), this.saturationStart.getValue(),
								this.valueStart.getValue());
						Scalar maxValues = new Scalar(this.hueStop.getValue(), this.saturationStop.getValue(),
								this.valueStop.getValue());
						
						// show the current selected HSV range
						String valuesToPrint = "Hue range: " + minValues.val[0] + "-" + maxValues.val[0]
								+ "\tSaturation range: " + minValues.val[1] + "-" + maxValues.val[1] + "\tValue range: "
								+ minValues.val[2] + "-" + maxValues.val[2];
						this.onFXThread(this.hsvValuesProp, valuesToPrint);
						
						// threshold HSV image to select tennis balls
						Core.inRange(hsvImage, minValues, maxValues, mask);
						// show the partial output
						this.onFXThread(maskProp, this.mat2Image(mask));
						
						// morphological operators
						// dilate with large element, erode with small ones
						Mat dilateElement = Imgproc.getStructuringElement(Imgproc.MORPH_RECT, new Size(24, 24));
						Mat erodeElement = Imgproc.getStructuringElement(Imgproc.MORPH_RECT, new Size(12, 12));
						
						Imgproc.erode(mask, morphOutput, erodeElement);
						Imgproc.erode(mask, morphOutput, erodeElement);
						
						Imgproc.dilate(mask, morphOutput, dilateElement);
						Imgproc.dilate(mask, morphOutput, dilateElement);
						
						// show the partial output
						this.onFXThread(this.morphProp, this.mat2Image(morphOutput));
						
						// find the tennis ball(s) contours and show them
						frame = this.findAndDrawBalls(morphOutput, frame);
						
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
		 * Given a binary image containing one or more closed surfaces, use it as a
		 * mask to find and highlight the objects contours
		 * 
		 * @param maskedImage
		 *            the binary image to be used as a mask
		 * @param frame
		 *            the original {@link Mat} image to be used for drawing the
		 *            objects contours
		 * @return the {@link Mat} image with the objects contours framed
		 */
		private Mat findAndDrawBalls(Mat maskedImage, Mat frame)
		{
			// init
			List<MatOfPoint> contours = new ArrayList<>();
			Mat hierarchy = new Mat();
			
			// find contours
			Imgproc.findContours(maskedImage, contours, hierarchy, Imgproc.RETR_CCOMP, Imgproc.CHAIN_APPROX_SIMPLE);
			
			// if any contour exist...
			if (hierarchy.size().height > 0 && hierarchy.size().width > 0)
			{
				// for each contour, display it in blue
				for (int idx = 0; idx >= 0; idx = (int) hierarchy.get(0, idx)[0])
				{
					Imgproc.drawContours(frame, contours, idx, new Scalar(250, 0, 0));
				}
			}
			
			return frame;
		}
		
		/**
		 * Set typical {@link ImageView} properties: a fixed width and the
		 * information to preserve the original image ration
		 * 
		 * @param image
		 *            the {@link ImageView} to use
		 * @param dimension
		 *            the width of the image to set
		 */
		private void imageViewProperties(ImageView image, int dimension)
		{
			// set a fixed width for the given ImageView
			image.setFitWidth(dimension);
			// preserve the image ratio
			image.setPreserveRatio(true);
		}
		
		/**
		 * Convert a {@link Mat} object (OpenCV) in the corresponding {@link Image}
		 * for JavaFX
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
		
		/**
		 * Generic method for putting element running on a non-JavaFX thread on the
		 * JavaFX thread, to properly update the UI
		 * 
		 * @param property
		 *            a {@link ObjectProperty}
		 * @param value
		 *            the value to set for the given {@link ObjectProperty}
		 */
		private <T> void onFXThread(final ObjectProperty<T> property, final T value)
		{
			Platform.runLater(new Runnable() {
				
				@Override
				public void run()
				{
					property.set(value);
				}
			});
		}
		
	}


- `ObjRecognition.fxml <https://github.com/opencv-java/object-detection/blob/master/src/it/polito/teaching/cv/ObjRecognition.fxml>`_

.. code-block:: xml


   <BorderPane xmlns:fx="http://javafx.com/fxml" fx:controller="it.polito.teaching.cv.ObjRecognitionController">
	<right>
		<VBox alignment="CENTER" spacing="10">
			<padding>
				<Insets right="10" left="10" />
			</padding>
			<Label text="Hue Start" />
			<Slider fx:id="hueStart" min="0" max="180" value="20" blockIncrement="1" />
			<Label text="Hue Stop" />
			<Slider fx:id="hueStop" min="0" max="180" value="50" blockIncrement="1" />
			<Label text="Saturation Start" />
			<Slider fx:id="saturationStart" min="0" max="255" value="60" blockIncrement="1" />
			<Label text="Saturation Stop" />
			<Slider fx:id="saturationStop" min="0" max="255" value="200" blockIncrement="1" />
			<Label text="Value Start" />
			<Slider fx:id="valueStart" min="0" max="255" value="50" blockIncrement="1" />
			<Label text="Value Stop" />
			<Slider fx:id="valueStop" min="0" max="255" value="255" blockIncrement="1" />
		</VBox>
	</right>
	<center>
		<HBox alignment="CENTER" spacing="5">
			<padding>
				<Insets right="10" left="10" />
			</padding>
			<ImageView fx:id="originalFrame" />
			<VBox alignment="CENTER" spacing="5">
				<ImageView fx:id="maskImage" />
				<ImageView fx:id="morphImage" />
			</VBox>
		</HBox>
	</center>
	<bottom>
		<VBox alignment="CENTER" spacing="15">
			<padding>
				<Insets top="25" right="25" bottom="25" left="25" />
			</padding>
			<Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" />
			<Separator />
			<Label fx:id="hsvCurrentValues" />
		</VBox>
	</bottom>
   </BorderPane>


