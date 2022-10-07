# GallerryApp


# Mianactivity Code:

package com.shah.galleryapp

import android.Manifest
import android.content.pm.PackageManager
import android.database.Cursor
import android.os.Build
import android.os.Bundle
import android.os.Environment
import android.provider.MediaStore
import android.util.Log
import androidx.annotation.RequiresApi
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import androidx.recyclerview.widget.GridLayoutManager
import com.shah.galleryapp.databinding.ActivityMainBinding
import java.util.*


class MainActivity : AppCompatActivity() {
    lateinit var binding : ActivityMainBinding
    private var allPictures:ArrayList<Images>? = null


    @RequiresApi(Build.VERSION_CODES.O)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Storage Permissions
        if ( ContextCompat.checkSelfPermission (this@MainActivity , Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions (this@MainActivity, arrayOf( Manifest.permission.READ_EXTERNAL_STORAGE ) ,  101)

        }

        allPictures = ArrayList()
        if(allPictures!!.isEmpty()){
            allPictures = getAllImages()
            Log.d("TAG-SIZE-1",allPictures!!.size.toString())

            //setting adapter
            binding.recView.layoutManager = GridLayoutManager(this,3)
            binding.recView.setHasFixedSize(true)
            binding.recView.adapter = ImagesAdapter(this,allPictures!!)
        }

    }

    @RequiresApi(Build.VERSION_CODES.O)
    private fun getAllImages(): ArrayList<Images>? {
        val images = ArrayList<Images>()

       //uri
        val uri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI

       //projection to take a target pics
        val projection = arrayOf(MediaStore.Images.ImageColumns.DATA, MediaStore.Images.Media.DISPLAY_NAME)

        val CAMERA_IMAGE_BUCKET_NAME = (Environment.getExternalStorageDirectory().toString() + "/DCIM/Camera")
        val CAMERA_IMAGE_BUCKET_ID: String = getBucketId(CAMERA_IMAGE_BUCKET_NAME)
        val selection = MediaStore.Images.Media.BUCKET_ID + " = ?"
        val selectionArgs = arrayOf<String>(CAMERA_IMAGE_BUCKET_ID)

        Log.d("TAG-BUCKET-Name",CAMERA_IMAGE_BUCKET_NAME)

      //cursor to get images from a bucket
        val cursor = this@MainActivity.contentResolver.query(uri,projection,null,null)

        try {
            cursor!!.moveToFirst()

            do {
                val image = Images()

                image.imgPath = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DATA))
                image.imgName = cursor.getString(cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME))
                images.add(image)
            }while(cursor.moveToNext())
            cursor.close()

        }catch (e:Exception){
            e.printStackTrace()
        }
        return images
    }

    fun getBucketId(path: String): String {
        return path.lowercase(Locale.getDefault()).hashCode().toString()
    }
}



# Model Code:

package com.shah.galleryapp

class Images {
    var imgName : String? = null
    var imgPath : String? = null

    constructor(imgName: String?, imgPath: String?) {
        this.imgName = imgName
        this.imgPath = imgPath
    }

    constructor(){}
}

# Adapter Code:

package com.shah.galleryapp

import android.content.Context
import android.util.Log
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.bumptech.glide.request.RequestOptions

class ImagesAdapter(private var context:Context,private var imgList : ArrayList<Images>) : RecyclerView.Adapter<ImagesAdapter.ImgViewHolder>() {


    class ImgViewHolder(itemView: View): RecyclerView.ViewHolder(itemView) {
        var imgView : ImageView?=null
        init {
            imgView = itemView.findViewById(R.id.imgView)
        }

    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ImgViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        val view = inflater.inflate(R.layout.single_item,parent,false)
        return ImgViewHolder(view)

    }

    override fun onBindViewHolder(holder: ImgViewHolder, position: Int) {
        val currentImage = imgList[position]

        Log.d("TAG-SIZE-2",imgList.size.toString())

        Glide.with(context).load(currentImage.imgPath)
            .apply(RequestOptions().centerCrop()).into(holder.imgView!!)


    }

    override fun getItemCount(): Int {
        return imgList.size
    }

}


