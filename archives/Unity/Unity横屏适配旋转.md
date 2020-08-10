void Awake()
	{
		//设置屏幕正方向在Home键右边
		Screen.orientation = ScreenOrientation.LandscapeRight;
	}
 
	void Start () 
	{
		//设置屏幕自动旋转， 并置支持的方向
		Screen.orientation = ScreenOrientation.AutoRotation;
		Screen.autorotateToLandscapeLeft = true;
		Screen.autorotateToLandscapeRight = true;
		Screen.autorotateToPortrait = false;
		Screen.autorotateToPortraitUpsideDown = false;
	}
