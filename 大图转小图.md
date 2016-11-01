

```
step1:获得低分辨率bitmap b1
step2:b1写入文件

使用场景:上传相册中的大图
```

```
 /**
 * @author shoyu666@163.com
 */
public class ImageUtil {
    private static final String TAG = "ImageUtil";

    /**
     * @param sourceUri 要压缩保存的图片
     * @param f         保存的目录
     * @param w         期望宽度
     * @param quality   质量
     * @return
     */
    public static boolean compressBitmap(Uri sourceUri, File f, int w,
                                         int quality) {
        Context context = MAppManager.getApplication();
        if (context == null) return false;
        try {
            int sample = getinSampleSize(sourceUri, context, w);
            if (sample == 1) {
                return false;
            }
            BitmapFactory.Options option = new BitmapFactory.Options();
            InputStream is = context.getContentResolver().openInputStream(
                    sourceUri);
            option.inSampleSize = sample;
            Bitmap bt = BitmapFactory.decodeStream(is, null, option);
            boolean bit = compressBitmap(bt, f, quality, null);
            is.close();
            return bit;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }


    /**
     * @param bitmapToSave
     * @param f
     * @param quality
     * @param source
     * @return
     */
    private static boolean compressBitmap(Bitmap bitmapToSave, File f,
                                          int quality, Bitmap source) {
        if (bitmapToSave == null) {
            Lx.e(TAG, "Bitmap  null");
            return false;
        }
        FileOutputStream out;
        BufferedOutputStream bos;
        Bitmap rotaBitmap = null;
        try {
            out = new FileOutputStream(f);
            bos = new BufferedOutputStream(out);
            boolean result;
            result = bitmapToSave.compress(CompressFormat.JPEG, quality,
                    bos);
            Lx.d(TAG, "rotaBitmap  h  " + bitmapToSave.getHeight() + "  w " + bitmapToSave.getWidth());
            bos.flush();
            bos.close();
            return result;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (bitmapToSave != null)
                bitmapToSave.recycle();
            if (source != null)
                source.recycle();
            if (rotaBitmap != null)
                rotaBitmap.recycle();
        }
        return false;
    }

    /**
     * @param sourceUri
     * @param context
     * @param w
     * @return
     */
    public static int getinSampleSize(Uri sourceUri, Context context, int w) {
        int inSampleSize = 1;
        try {
            BitmapFactory.Options option = new BitmapFactory.Options();
            option.inJustDecodeBounds = true;
            InputStream is = context.getContentResolver().openInputStream(
                    sourceUri);
            BitmapFactory.decodeStream(is, null, option);
            int bitmapWidth = option.outWidth;
            int bitmapHeight = option.outHeight;
            inSampleSize = sampleSize(bitmapWidth, bitmapHeight, w);
            Lx.d(TAG, "getinSampleSize inSampleSize " + inSampleSize
                    + " bitmapWidth " + bitmapWidth + " bitmapHeight " + bitmapHeight);
            return inSampleSize;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return inSampleSize;
    }

    public static int sampleSize(int width, int height, int target) {
        if (width > height) width = height;
        int result = 1;
        for (int i = 0; i < 10; i++) {
            if (width < target * 2) {
                break;
            }
            width = width / 2;
            result = result * 2;
        }
        return result;
    }
}
```
