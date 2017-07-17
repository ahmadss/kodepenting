    public class MainActivity extends AppCompatActivity {

    private ListView lstText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        // Create default options which will be used for every
    //  displayImage(...) call if no options will be passed to this method
        DisplayImageOptions defaultOptions = new DisplayImageOptions.Builder().cacheInMemory(true)
                .cacheOnDisk(true).build();
        ImageLoaderConfiguration config = new       ImageLoaderConfiguration.Builder(getApplicationContext()).defaultDisplayImageOptions(defaultOptions).build();
        ImageLoader.getInstance().init(config); // Do it on Application start

        // Default options will be used
        // DisplayImageOptions options = new DisplayImageOptions.Builder()
        // .cacheInMemory(true)
        // .cacheOnDisk(true)
        // .build();

        lstText = (ListView)findViewById(R.id.listMovies);


        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                new JsonTask().execute("http://jsonparsing.parseapp.com/jsonData/moviesData.txt");
            }
        });
    }

    public class JsonTask extends AsyncTask<String, String, List<MovieModel>> {

        @Override
        protected List<MovieModel> doInBackground(String... params) {
            HttpURLConnection connection = null;
            BufferedReader bufferedReader = null;

            try {
                URL url = new URL(params[0]);
                connection = (HttpURLConnection) url.openConnection();
                connection.connect();

                InputStream stream = connection.getInputStream();
                bufferedReader = new BufferedReader(new InputStreamReader(stream));

                StringBuffer stringBuffer = new StringBuffer();
                String line = "";
                while ((line = bufferedReader.readLine()) != null) {
                    stringBuffer.append(line);
                }

    //                {
    //                    "movies": [
    //                    {
    //                        "movie": "Avengers",
    //                            "year": 2012
    //                    }
    //                    ]
    //                }
                String finalJson = stringBuffer.toString();
                JSONObject parentJosn = new JSONObject(finalJson);

                JSONArray parentArray = parentJosn.getJSONArray("movies");

                List<MovieModel> movieModelList = new ArrayList<>();
    //          StringBuffer finalBuffer = new StringBuffer();
                for (int i = 0; i < parentArray.length(); i++) {
                    JSONObject finalObjek = parentArray.getJSONObject(i);
                    MovieModel movieModel = new MovieModel();
                    movieModel.setMovie(finalObjek.getString("movie"));

                    movieModel.setImage(finalObjek.getString("image"));
                    movieModel.setStory(finalObjek.getString("story"));

                    List<MovieModel.Cast> castList = new ArrayList<>();
                    for (int j = 0; j<finalObjek.getJSONArray("cast").length(); j++){
                        JSONObject castObjek = finalObjek.getJSONArray("cast").getJSONObject(j);
                        MovieModel.Cast cast = new MovieModel.Cast();
                        cast.setName(finalObjek.getJSONArray("cast").getJSONObject(j).getString("name"));
                        castList.add(cast);
                    }

                    movieModel.setCastList(castList);

                    //menambah ke final objek
                    movieModelList.add(movieModel);

                    String nama = finalObjek.getString("movie");
                    int tahun = finalObjek.getInt("year");
                    finalBuffer.append(nama + " - " + tahun + "\n");
                }

                return movieModelList;

            } catch (IOException e) {
                e.printStackTrace();
            } catch (JSONException e) {
                e.printStackTrace();
            } finally {
                if (connection != null) {
                    connection.disconnect();
                }
                try {
                    if (bufferedReader != null) {
                        bufferedReader.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            return null;
        }

        @Override
        protected void onPostExecute(List<MovieModel> result) {
            super.onPostExecute(result);
            jsnText.setText(result);
            //SET DATA UNTUL VIEWLIST
            MovieAdapter adapter = new MovieAdapter(getApplicationContext(), R.layout.row_list, result);
            lstText.setAdapter(adapter);
        }
    }

    public class MovieAdapter extends ArrayAdapter{

        public List<MovieModel> movieModelList;
        private int resource;
        private LayoutInflater inflater;

        public MovieAdapter(Context context, int resource, List<MovieModel> objects) {
            super(context, resource, objects);
            movieModelList = objects;
            this.resource = resource;
            inflater = (LayoutInflater) getSystemService(LAYOUT_INFLATER_SERVICE);
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {

            ViewHolder holder = null;

            if (convertView == null){
                holder = new ViewHolder();
                convertView = inflater.inflate(resource, null);
                holder.imageViewIcon = (ImageView)convertView.findViewById(R.id.iconImg);
                holder.textMovie = (TextView)convertView.findViewById(R.id.MovieName);
                holder.textTagline = (TextView)convertView.findViewById(R.id.textTagLine);
                holder.textYear = (TextView)convertView.findViewById(R.id.textYear);
                holder.textDuration = (TextView)convertView.findViewById(R.id.textDuration);
                holder.textDirector = (TextView)convertView.findViewById(R.id.textDirectur);
                holder.rbMovieRating = (RatingBar)convertView.findViewById(R.id.ratingBar);
                holder.textCast = (TextView)convertView.findViewById(R.id.textCast);
                holder.textStory = (TextView)convertView.findViewById(R.id.textStory);
                convertView.setTag(holder);
            } else {
                holder = (ViewHolder) convertView.getTag();
            }


            final ProgressBar progressBar = (ProgressBar)convertView.findViewById(R.id.progressBar);

            // Then later, when you want to display image
            ImageLoader.getInstance().displayImage(movieModelList.get(position).getImage(), holder.imageViewIcon, new ImageLoadingListener() {
                @Override
                public void onLoadingStarted(String imageUri, View view) {
                    progressBar.setVisibility(View.VISIBLE);
                }

                @Override
                public void onLoadingFailed(String imageUri, View view, FailReason failReason) {
                    progressBar.setVisibility(View.GONE);
                }

                @Override
                public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
                    progressBar.setVisibility(View.GONE);
                }

                @Override
                public void onLoadingCancelled(String imageUri, View view) {
                    progressBar.setVisibility(View.GONE);
                }
            });

            holder.textMovie.setText(movieModelList.get(position).getMovie());
            holder.textTagline.setText(movieModelList.get(position).getTagline());
            holder.textYear.setText("Year: "+movieModelList.get(position).getYear());
            holder.textDuration.setText(movieModelList.get(position).getDuration());
            holder.textDirector.setText(movieModelList.get(position).getDirector());
            holder.rbMovieRating.setRating((int)movieModelList.get(position).getRating()/2);
    //      Log.d("Rating", "getView: "+movieModelList.get(position).getRating()/2);

            StringBuffer stringBuffer = new StringBuffer();
            for (MovieModel.Cast cast : movieModelList.get(position).getCastList()){
                stringBuffer.append(cast.getName() + ", ");
            }

            holder.textCast.setText(stringBuffer);
            holder.textStory.setText(movieModelList.get(position).getStory());


            return convertView;
        }

        class ViewHolder {
            ImageView imageViewIcon;
            TextView textMovie;
            TextView textTagline;
            TextView textYear;
            TextView textDuration;
            TextView textDirector;
            RatingBar rbMovieRating;
            TextView textCast;
            TextView textStory;
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_refresh) {
            new JsonTask().execute("http://jsonparsing.parseapp.com/jsonData/moviesData.txt");
            return true;
        }

        return super.onOptionsItemSelected(item);
    }
    
    public class MovieModel {

    private String movie;
    private int year;
    private float rating;
    private String duration;
    private String director;
    private String tagline;
    //cast
    private List<Cast> castList;
    private String image;
    private String story;

    public String getMovie() {
        return movie;
    }

    public void setMovie(String movie) {
        this.movie = movie;
    }

    public int getYear() {
        return year;
    }

    public void setYear(int year) {
        this.year = year;
    }

    public float getRating() {
        return rating;
    }

    public void setRating(float rating) {
        this.rating = rating;
    }

    public String getDuration() {
        return duration;
    }

    public void setDuration(String duration) {
        this.duration = duration;
    }

    public String getDirector() {
        return director;
    }

    public void setDirector(String director) {
        this.director = director;
    }

    public String getTagline() {
        return tagline;
    }

    public void setTagline(String tagline) {
        this.tagline = tagline;
    }

    public List<Cast> getCastList() {
        return castList;
    }

    public void setCastList(List<Cast> castList) {
        this.castList = castList;
    }

    public String getImage() {
        return image;
    }

    public void setImage(String image) {
        this.image = image;
    }

    public String getStory() {
        return story;
    }

    public void setStory(String story) {
        this.story = story;
    }

    public static class Cast{
        private String name;

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
}
