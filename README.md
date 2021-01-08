# UPLOAD_MULT_IMAGE_TO_FIREBASE_STORAGE+LINK

Okey this easy thing to do listen copy/past this code and run your code :)


```sh
public class testMultImageFirebase extends AppCompatActivity {

    private static final int PICK_IMG = 1;
    private ArrayList<Uri> ImageList = new ArrayList<Uri>();
    private DatabaseReference databaseReference;
    private ProgressDialog progressDialog;
    TextView textView;
    Button choose, send;
    Map<String,String> imageStringList;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test_mult_image_firebase);

        databaseReference = FirebaseDatabase.getInstance().getReference().child("User");
        imageStringList = new HashMap<>();
        progressDialog = new ProgressDialog(this);
        progressDialog.setMessage("Uploading ..........");
        textView = findViewById(R.id.text);
        choose = findViewById(R.id.choose);
        send = findViewById(R.id.upload);
        choose.setOnClickListener(v -> choose());
        send.setOnClickListener(v -> upload());
    }

    public void choose() {
        //we will pick images
        Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.setType("image/*");
        intent.putExtra(Intent.EXTRA_ALLOW_MULTIPLE, true);
        startActivityForResult(intent, PICK_IMG);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == PICK_IMG) {
            if (resultCode == RESULT_OK) {
                if (data.getClipData() != null) {
                    int count = data.getClipData().getItemCount();

                    int CurrentImageSelect = 0;

                    while (CurrentImageSelect < count) {
                        Uri imageuri = data.getClipData().getItemAt(CurrentImageSelect).getUri();
                        ImageList.add(imageuri);
                        CurrentImageSelect = CurrentImageSelect + 1;
                    }
                    textView.setVisibility(View.VISIBLE);
                    textView.setText("You Have Selected " + ImageList.size() + " Pictures");
                    choose.setVisibility(View.GONE);
                }

            }

        }

    }

    @SuppressLint("SetTextI18n")
    public void upload() {

        textView.setText("Please Wait ... If Uploading takes Too much time please the button again ");
        progressDialog.show();
        final StorageReference ImageFolder = FirebaseStorage.getInstance().getReference().child("ImageFolder");
        int uploads = 0;
        for (uploads = 0; uploads < ImageList.size(); uploads++) {
            Uri Image = ImageList.get(uploads);
            final StorageReference imageName = ImageFolder.child("image/" + Image.getLastPathSegment());

            imageName.putFile(ImageList.get(uploads))
                    .addOnFailureListener(Throwable::printStackTrace)
                    .addOnSuccessListener(taskSnapshot ->
                            imageName.getDownloadUrl().addOnSuccessListener(uri -> {
                                String url = String.valueOf(uri);
                                imageStringList.put("image",url);
                                SendLink(imageStringList);
                                System.out.println(imageStringList.size());
                            }));
        }


    }

    private void SendLink(Map<String, String> url) {
        databaseReference
                .child("images")
                .push()
                .setValue(url)
                .addOnFailureListener(Throwable::printStackTrace)
                .addOnCompleteListener(task -> {
                    progressDialog.dismiss();
                    textView.setText("Image Uploaded Successfully");
                    send.setVisibility(View.GONE);
                    ImageList.clear();
                });
    }
}

```
