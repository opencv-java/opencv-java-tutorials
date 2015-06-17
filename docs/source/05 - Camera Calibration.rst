==================
Camera Calibration
==================

.. note:: We assume that by now you have already read the previous tutorials. If not, please check previous tutorials at `<http://opencv-java-tutorials.readthedocs.org/en/latest/index.html>`_. You can also find the source code and resources at `<https://github.com/opencv-java/>`_

Goal
----
The goal of this tutorial is to learn how to calibrate a camera given a set of chessboard images.

What is the camera calibration?
-------------------------------
The camera calibration is the process with which we can obtain the camera parameters such as intrinsic and extrinsic parameters, distortions and so on. The calibration of the camera is often necessary when the alignment between the lens and the optic sensors chip is not correct; the effect produced by this wrong alignment is usually more visible in low quality cameras.

Calibration Pattern
-------------------
As we said earlier we are going to need some sort of pattern that the program can recognize in order to make the calibration work. The pattern that we are going to use is a chessboard image.

.. image:: _static/05-00.png

The reason why we use this image is because there are some OpenCV functions that can recognize this pattern and draw a scheme which highlights the intersections between each block.
To make the calibration work you need to print the chessboard image and show it to the cam; it is important to maintain the sheet still, better if stick to a surface.
In order to make a good calibration, we need to have about 20 samples of the pattern taken from different angles and distances.

What we will do in this tutorial
--------------------------------
In this guide, we will:
 * Create some TextEdit field to give some inputs to our program
 * Recognize the pattern using some OpenCV functions
 * Calibrate and show the video stream.

Getting Started
---------------
Create a new JavaFX project (e.g. "CameraCalibration") with the usual OpenCV user library.
Open Scene Builder and add a Border Pane with:

- on **TOP** we need to have the possibility to set the number of samples for the calibration, the number of horizontal corners we have in the test image, the number of vertical corners we have in the test image and a button to update this data. To make things cleaner let's put all these elements inside a HBox.

.. code-block:: xml

    <HBox alignment="CENTER" spacing="10">

Let's also add some labels before each text fields.
Each text field is going to need an id, and let's put a standard value for them already.

.. code-block:: xml

    <Label text="Boards #" />
    <TextField fx:id="numBoards" text="20" maxWidth="50" />
    <Label text="Horizontal corners #" />
    <TextField fx:id="numHorCorners" text="9" maxWidth="50" />
    <Label text="Vertical corners #" />
    <TextField fx:id="numVertCorners" text="6" maxWidth="50" />

For the button instead, set the id and a method for the onAction field:

.. code-block:: xml

    <Button fx:id="applyButton" alignment="center" text="Apply" onAction="#updateSettings" />

- on the **LEFT** add an ImageView inside a VBox for the normal cam stream; set an id for it.

.. code-block:: xml

    <ImageView fx:id="originalFrame" />

- on the **RIGHT** add an ImageView inside a VBox for the calibrated cam stream; set an id for it.

.. code-block:: xml

    <ImageView fx:id="originalFrame" />

- in the **BOTTOM** add a start/stop cam stream button and a snapshot button inside a HBox; set an id and a action method for each one.

.. code-block:: xml

    <Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" disable="true" />
    <Button fx:id="snapshotButton" alignment="center" text="Take snapshot" onAction="#takeSnapshot" disable="true" />

Your GUI will look something like this:

.. image:: _static/05-03.png

Pattern Recognition
-------------------
The calibration process consists on showing to the cam the chessboard pattern from different angles, depth and points of view. For each recognized pattern we need to track:

 - some reference system's 3D point where the chessboard is located (let's assume that the Z axe is always 0):

	.. code-block:: java

		for (int j = 0; j < numSquares; j++)
		   obj.push_back(new MatOfPoint3f(new Point3(j / this.numCornersHor, j % this.numCornersVer, 0.0f)));

 - the image's 2D points (operation made by OpenCV with findChessboardCorners):

	.. code-block:: java

		boolean found = Calib3d.findChessboardCorners(grayImage, boardSize, imageCorners, Calib3d.CALIB_CB_ADAPTIVE_THRESH + Calib3d.CALIB_CB_NORMALIZE_IMAGE + Calib3d.CALIB_CB_FAST_CHECK);

The ``findChessboardCorners`` function attempts to determine whether the input image is a view of the chessboard pattern and locate the internal chessboard corners.
Its parameters are:

 - **image** Source chessboard view. It must be an 8-bit grayscale or color image.
 - **patternSize** Number of inner corners per a chessboard row and column
 - **corners** Output array of detected corners.
 - **flags** Various operation flags that can be zero or a combination of the following values:
	- ``CV_CALIB_CB_ADAPTIVE_THRESH`` Use adaptive thresholding to convert the image to black and white, rather than a fixed threshold level (computed from the average image brightness).
	- ``CV_CALIB_CB_NORMALIZE_IMAGE`` Normalize the image gamma with "equalizeHist" before applying fixed or adaptive thresholding.
	- ``CV_CALIB_CB_FILTER_QUADS`` Use additional criteria (like contour area, perimeter, square-like shape) to filter out false quads extracted at the contour retrieval stage.
	- ``CALIB_CB_FAST_CHECK`` Run a fast check on the image that looks for chessboard corners, and shortcut the call if none is found. This can drastically speed up the call in the degenerate condition when no chessboard is observed.

.. warning:: Before doing the ``findChessboardCorners`` convert the image to gayscale and save the board size into a Size variable:

	.. code-block:: java

	    Imgproc.cvtColor(frame, grayImage, Imgproc.COLOR_BGR2GRAY);
	    Size boardSize = new Size(this.numCornersHor, this.numCornersVer);

If the recognition went well ``found`` should be ``true``.

For square images the positions of the corners are only approximate. We may improve this by calling the ``cornerSubPix`` function. It will produce better calibration result.

.. code-block:: java

    TermCriteria term = new TermCriteria(TermCriteria.EPS | TermCriteria.MAX_ITER, 30, 0.1);
    Imgproc.cornerSubPix(grayImage, imageCorners, new Size(11, 11), new Size(-1, -1), term);

We can now highlight the found points on stream:

.. code-block:: java

    Calib3d.drawChessboardCorners(frame, boardSize, imageCorners, found);

The function draws individual chessboard corners detected either as red circles if the board was not found, or as colored corners connected with lines if the board was found.

Its parameters are:

 - **image** Destination image. It must be an 8-bit color image.
 - **patternSize** Number of inner corners per a chessboard row and column.
 - **corners** Array of detected corners, the output of findChessboardCorners.
 - **patternWasFound** Parameter indicating whether the complete board was found or not. The return value of ``findChessboardCorners`` should be passed here.

Now we can activate the Snapshot button to save the data.

.. code-block:: java

    this.snapshotButton.setDisable(false);

.. image:: _static/05-01.png

.. image:: _static/05-02.png

We should take the set number of "snapshots" from different angles and depth, in order to make the calibration.

.. note:: We don't actually save the image but just the data we need.

Saving Data
-----------
By clicking on the snapshot button we cal the ``takeSnapshot`` method. Here we need to save the data (2D and 3D points)  if we did not make enough sample:

.. code-block:: java

    this.imagePoints.add(imageCorners);
    this.objectPoints.add(obj);
    this.successes++;

Otherwise we can calibrate the camera.

Camera Calibration
------------------
For the camera calibration we should create initiate some needed variable and then call the actual calibration function:

.. code-block:: java

    List<Mat> rvecs = new ArrayList<>();
    List<Mat> tvecs = new ArrayList<>();
    intrinsic.put(0, 0, 1);
    intrinsic.put(1, 1, 1);

    Calib3d.calibrateCamera(objectPoints, imagePoints, savedImage.size(), intrinsic, distCoeffs, rvecs, tvecs);

The ``calibrateCamera`` function estimates the intrinsic camera parameters and extrinsic parameters for each of the views. The algorithm is based on [Zhang2000] and [BouguetMCT]. The coordinates of 3D object points and their corresponding 2D projections in each view must be specified.
Its parameters are:

 - **objectPoints** In the new interface it is a vector of vectors of calibration pattern points in the calibration pattern coordinate space. The outer vector contains as many elements as the number of the pattern views. The points are 3D, but since they are in a pattern coordinate system, then, if the rig is planar, it may make sense to put the model to a XY coordinate plane so that Z-coordinate of each input object point is 0.
 - **imagePoints** It is a vector of vectors of the projections of calibration pattern points.
 - **imageSize** Size of the image used only to initialize the intrinsic camera matrix.
 - **cameraMatrix** Output 3x3 floating-point camera matrix *A = |fx 0 cx| |0 fy cy| |0 0 1|*. If ``CV_CALIB_USE_INTRINSIC_GUESS`` and/or ``CV_CALIB_FIX_ASPECT_RATIO`` are specified, some or all of *fx*, *fy*, *cx*, *cy* must be initialized before calling the function.
 - **distCoeffs** Output vector of distortion coefficients of 4, 5, or 8 elements.
 - **rvecs** Output vector of rotation vectors estimated for each pattern view. That is, each k-th rotation vector together with the corresponding k-th translation vector.
 - **tvecs** Output vector of translation vectors estimated for each pattern view.

We ran calibration and got camera's matrix with the distortion coefficients we may want to correct the image using ``undistort`` function:

.. code-block:: java

    if (this.isCalibrated)
    {
	// prepare the undistored image
	Mat undistored = new Mat();
	Imgproc.undistort(frame, undistored, intrinsic, distCoeffs);
	undistoredImage = mat2Image(undistored);
    }

The ``undistort`` function transforms an image to compensate radial and tangential lens distortion.

Source Code
-----------
- `CameraCalibration.java <https://github.com/opencv-java/camera-calibration/blob/master/src/application/CameraCalibration.java>`_

.. code-block:: java

    public class CameraCalibration extends Application {
	@Override
	public void start(Stage primaryStage) {
		try {
			// load the FXML resource
			FXMLLoader loader = new FXMLLoader(getClass().getResource("CC_FX.fxml"));
			// store the root element so that the controllers can use it
			BorderPane rootElement = (BorderPane) loader.load();
			// set a whitesmoke background
			rootElement.setStyle("-fx-background-color: whitesmoke;");
			// create and style a scene
			Scene scene = new Scene(rootElement, 800, 600);
			scene.getStylesheets().add(getClass().getResource("application.css").toExternalForm());
			// create the stage with the given title and the previously created
			// scene
			primaryStage.setTitle("Camera Calibration");
			primaryStage.setScene(scene);
			// init the controller variables
			CC_Controller controller = loader.getController();
			controller.init();
			// show the GUI
			primaryStage.show();
		} catch(Exception e) {
			e.printStackTrace();
		}
	}

	public static void main(String[] args) {
		// load the native OpenCV library
		System.loadLibrary(Core.NATIVE_LIBRARY_NAME);

		launch(args);
	}
    }

- `CC_Controller.java <https://github.com/opencv-java/camera-calibration/blob/master/src/application/CC_Controller.java>`_

.. code-block:: java

    public class CC_Controller {
	// FXML buttons
		@FXML
		private Button cameraButton;
		@FXML
		private Button applyButton;
		@FXML
		private Button snapshotButton;
		// the FXML area for showing the current frame (before calibration)
		@FXML
		private ImageView originalFrame;
		// the FXML area for showing the current frame (after calibration)
		@FXML
		private ImageView calibratedFrame;
		// info related to the calibration process
		@FXML
		private TextField numBoards;
		@FXML
		private TextField numHorCorners;
		@FXML
		private TextField numVertCorners;

		// a timer for acquiring the video stream
		private Timer timer;
		// the OpenCV object that performs the video capture
		private VideoCapture capture;
		// a flag to change the button behavior
		private boolean cameraActive;
		// the saved chessboard image
		private Mat savedImage;
		// the calibrated camera frame
		private Image undistoredImage,CamStream;
		// various variables needed for the calibration
		private List<Mat> imagePoints;
		private List<Mat> objectPoints;
		private MatOfPoint3f obj;
		private MatOfPoint2f imageCorners;
		private int boardsNumber;
		private int numCornersHor;
		private int numCornersVer;
		private int successes;
		private Mat intrinsic;
		private Mat distCoeffs;
		private boolean isCalibrated;

		/**
		 * Init all the (global) variables needed in the controller
		 */
		protected void init()
		{
			this.capture = new VideoCapture();
			this.cameraActive = false;
			this.obj = new MatOfPoint3f();
			this.imageCorners = new MatOfPoint2f();
			this.savedImage = new Mat();
			this.undistoredImage = null;
			this.imagePoints = new ArrayList<>();
			this.objectPoints = new ArrayList<>();
			this.intrinsic = new Mat(3, 3, CvType.CV_32FC1);
			this.distCoeffs = new Mat();
			this.successes = 0;
			this.isCalibrated = false;
		}

		/**
		 * Store all the chessboard properties, update the UI and prepare other
		 * needed variables
		 */
		@FXML
		protected void updateSettings()
		{
			this.boardsNumber = Integer.parseInt(this.numBoards.getText());
			this.numCornersHor = Integer.parseInt(this.numHorCorners.getText());
			this.numCornersVer = Integer.parseInt(this.numVertCorners.getText());
			int numSquares = this.numCornersHor * this.numCornersVer;
			for (int j = 0; j < numSquares; j++)
				obj.push_back(new MatOfPoint3f(new Point3(j / this.numCornersHor, j % this.numCornersVer, 0.0f)));
			this.cameraButton.setDisable(false);
		}

		/**
		 * The action triggered by pushing the button on the GUI
		 */
		@FXML
		protected void startCamera()
		{
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
							CamStream=grabFrame();
							// show the original frames
							Platform.runLater(new Runnable() {
								@Override
					            public void run() {
									originalFrame.setImage(CamStream);
									// set fixed width
									originalFrame.setFitWidth(380);
									// preserve image ratio
									originalFrame.setPreserveRatio(true);
									// show the original frames
									calibratedFrame.setImage(undistoredImage);
									// set fixed width
									calibratedFrame.setFitWidth(380);
									// preserve image ratio
									calibratedFrame.setPreserveRatio(true);
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
					System.err.println("Impossible to open the camera connection...");
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
				// clean the image areas
				originalFrame.setImage(null);
				calibratedFrame.setImage(null);
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
						// show the chessboard pattern
						this.findAndDrawPoints(frame);

						if (this.isCalibrated)
						{
							// prepare the undistored image
							Mat undistored = new Mat();
							Imgproc.undistort(frame, undistored, intrinsic, distCoeffs);
							undistoredImage = mat2Image(undistored);
						}

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
		 * Take a snapshot to be used for the calibration process
		 */
		@FXML
		protected void takeSnapshot()
		{
			if (this.successes < this.boardsNumber)
			{
				// save all the needed values
				this.imagePoints.add(imageCorners);
				this.objectPoints.add(obj);
				this.successes++;
			}

			// reach the correct number of images needed for the calibration
			if (this.successes == this.boardsNumber)
			{
				this.calibrateCamera();
			}
		}

		/**
		 * Find and draws the points needed for the calibration on the chessboard
		 *
		 * @param frame
		 *            the current frame
		 * @return the current number of successfully identified chessboards as an
		 *         int
		 */
		private void findAndDrawPoints(Mat frame)
		{
			// init
			Mat grayImage = new Mat();

			// I would perform this operation only before starting the calibration
			// process
			if (this.successes < this.boardsNumber)
			{
				// convert the frame in gray scale
				Imgproc.cvtColor(frame, grayImage, Imgproc.COLOR_BGR2GRAY);
				// the size of the chessboard
				Size boardSize = new Size(this.numCornersHor, this.numCornersVer);
				// look for the inner chessboard corners
				boolean found = Calib3d.findChessboardCorners(grayImage, boardSize, imageCorners,
						Calib3d.CALIB_CB_ADAPTIVE_THRESH + Calib3d.CALIB_CB_NORMALIZE_IMAGE + Calib3d.CALIB_CB_FAST_CHECK);
				// all the required corners have been found...
				if (found)
				{
					// optimization
					TermCriteria term = new TermCriteria(TermCriteria.EPS | TermCriteria.MAX_ITER, 30, 0.1);
					Imgproc.cornerSubPix(grayImage, imageCorners, new Size(11, 11), new Size(-1, -1), term);
					// save the current frame for further elaborations
					grayImage.copyTo(this.savedImage);
					// show the chessboard inner corners on screen
					Calib3d.drawChessboardCorners(frame, boardSize, imageCorners, found);

					// enable the option for taking a snapshot
					this.snapshotButton.setDisable(false);
				}
				else
				{
					this.snapshotButton.setDisable(true);
				}
			}
		}

		/**
		 * The effective camera calibration, to be performed once in the program
		 * execution
		 */
		private void calibrateCamera()
		{
			// init needed variables according to OpenCV docs
			List<Mat> rvecs = new ArrayList<>();
			List<Mat> tvecs = new ArrayList<>();
			intrinsic.put(0, 0, 1);
			intrinsic.put(1, 1, 1);
			// calibrate!
			Calib3d.calibrateCamera(objectPoints, imagePoints, savedImage.size(), intrinsic, distCoeffs, rvecs, tvecs);
			this.isCalibrated = true;

			// you cannot take other snapshot, at this point...
			this.snapshotButton.setDisable(true);
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

- `CC_FX.fxml <https://github.com/opencv-java/camera-calibration/blob/master/src/application/CC_FX.fxml>`_

.. code-block:: xml

    <BorderPane xmlns:fx="http://javafx.com/fxml/1" fx:controller="application.CC_Controller">
	<top>
		<VBox>
			<HBox alignment="CENTER" spacing="10">
				<padding>
					<Insets top="10" bottom="10" />
				</padding>
				<Label text="Boards #" />
				<TextField fx:id="numBoards" text="20" maxWidth="50" />
				<Label text="Horizontal corners #" />
				<TextField fx:id="numHorCorners" text="9" maxWidth="50" />
				<Label text="Vertical corners #" />
				<TextField fx:id="numVertCorners" text="6" maxWidth="50" />
				<Button fx:id="applyButton" alignment="center" text="Apply" onAction="#updateSettings" />
			</HBox>
			<Separator />
		</VBox>
	</top>
	<left>
		<VBox alignment="CENTER">
			<padding>
				<Insets right="10" left="10" />
			</padding>
			<ImageView fx:id="originalFrame" />
		</VBox>
	</left>
	<right>
		<VBox alignment="CENTER">
			<padding>
				<Insets right="10" left="10" />
			</padding>
			<ImageView fx:id="calibratedFrame" />
		</VBox>
	</right>
	<bottom>
		<HBox alignment="CENTER">
			<padding>
				<Insets top="25" right="25" bottom="25" left="25" />
			</padding>
			<Button fx:id="cameraButton" alignment="center" text="Start camera" onAction="#startCamera" disable="true" />
			<Button fx:id="snapshotButton" alignment="center" text="Take snapshot" onAction="#takeSnapshot" disable="true" />
		</HBox>
	</bottom>
    </BorderPane>
