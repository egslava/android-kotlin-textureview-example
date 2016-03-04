# Android Kotlin Textureview Example
How to use TextureView with OpenGL in your Android project? Example written in Kotlin! :-) A very simple example is about how to initialize OpenGL context on Android.

This example is not about stability, but about simplicity so you could overview and understand the whole process of OpenGL context initialization under Android.

This minimalistic example requires less than 80 lines of Kotlin code:
```Kotlin
class MainActivity : AppCompatActivity() {


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        texture_view.surfaceTextureListener = RendererThread()
    }

    inner class RendererThread : Thread(), TextureView.SurfaceTextureListener {
        override fun onSurfaceTextureUpdated(surface: SurfaceTexture?) { }
        override fun onSurfaceTextureSizeChanged(surface: SurfaceTexture?, width: Int, height: Int) { }
        override fun onSurfaceTextureDestroyed(surface: SurfaceTexture?) =  true
        override fun onSurfaceTextureAvailable(surface: SurfaceTexture?, width: Int, height: Int) = start()

        val config = intArrayOf(
            EGL_RENDERABLE_TYPE, EGL_OPENGL_ES2_BIT,
            EGL_RED_SIZE, 8,
            EGL_GREEN_SIZE, 8,
            EGL_BLUE_SIZE, 8,
            EGL_ALPHA_SIZE, 8,
            EGL_DEPTH_SIZE, 0,
            EGL_STENCIL_SIZE, 0,
            EGL_NONE
        )

        fun  chooseEglConfig(egl: EGL10, eglDisplay: EGLDisplay) : EGLConfig {
            val configsCount = intArrayOf(0);
            val configs = arrayOfNulls<EGLConfig>(1);
            egl.eglChooseConfig(eglDisplay, config, configs, 1, configsCount)
            return configs[0]!!
        }

        override fun run() {
            super.run()

            val egl = EGLContext.getEGL() as EGL10
            val eglDisplay = egl.eglGetDisplay(EGL_DEFAULT_DISPLAY)
            egl.eglInitialize(eglDisplay, intArrayOf(0,0))   // getting OpenGL ES 2
            val eglConfig = chooseEglConfig(egl, eglDisplay);
            val eglContext = egl.eglCreateContext(eglDisplay, eglConfig, EGL_NO_CONTEXT, intArrayOf(EGL_CONTEXT_CLIENT_VERSION, 2, EGL_NONE));
            val eglSurface = egl.eglCreateWindowSurface(eglDisplay, eglConfig, texture_view.surfaceTexture, null)
            egl.eglMakeCurrent(eglDisplay, eglSurface, eglSurface, eglContext)

            var colorVelocity = 0.01f
            var color = 0f
            while(true){
                if (color > 1 || color < 0)colorVelocity *= -1
                color += colorVelocity

                GLES20.glClearColor(color / 2, color, color, 1.0f)
                GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT)
                egl.eglSwapBuffers(eglDisplay, eglSurface)
                Thread.sleep( (1f/60f * 1000f).toLong())
            }
        }
    }
}
```

If you want more - just download the source code.
