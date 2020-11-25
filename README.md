# Amazon-S3-Bucket-Laravel

  Amazon S3 Bucket Installation and Configuration using Laravel

  Step : 1 Installation

             In this step we have to install flysystem-aws-s3-v3 package for amazon api access.
             For Laravel they provide a package given below,
                
              composer require league/flysystem-aws-s3-v3

  Step : 2 Configuration

              After installation of package this will generate a file inside config directory
              called filesystems.php file

              In that file need to add S3 credentials in a disk array like shown below

                disks => [
     
                      's3' => [
                            'driver' => 's3',
                            'key'    => 'your-key',
                            'secret' => 'your-secret',
                            'region' => 'your-region',
                            'bucket' => 'your-bucket',
                        ],

  Step  : 3 Upload to S3

    /** upload audio stored in amazon s3
     *
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function uploadAudio(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'language_id' => 'required',
            'main_category_id' => 'required',
            'sub_category_id' => 'required',
            'title' => 'required',
            'audio' => 'required|mimes:application/octet-stream,audio/mpeg,mpga,mp3,wav',
        ]);
        if ($validator->fails())
        {
            $errorResponse = implode(',',$validator->errors()->all());
            return response()->json(['message'=>$errorResponse,'status'=>false]);
        }

        $authId =  $request->header('authId');

        if($request->hasfile('audio'))
        {
            $file = $request->file('audio');
            $name=time().$file->getClientOriginalName();
            $filePath = $authId.'/' . $name;
            Storage::disk('s3')->put($filePath, file_get_contents($file));
        }else{
            return response()->json(['message' => 'upload proper audio file', 'status' => false]);
        }

        $request = $request->except('audio');

        $this->audio->create(array_merge($request, ['audio' => Storage::disk('s3')->url($filePath),'user_id'=>$authId]));

        return response()->json(['message'=>'audio posted successfully','status'=>true]);

    }
