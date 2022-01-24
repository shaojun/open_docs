### rename `jpeg` to `jpg` files  **in place** :


`cvdata_convert --images_dir image_2/ --in_format jpeg --out_format jpg`

### convert `png` to `jpg` files  **in place** :


`cvdata_convert --images_dir image_2/ --in_format png --out_format jpg`

### resize `kitti` images and label files to a  **new folder** :


`cvdata_resize --input_images image_2/ --output_images resized_image_2 --input_annotations label_2/ --output_annotations resized_label_2 --width 800 --height 608 --format kitti
`

### rename image and label files name for a `kitti` dataset  **in place** :


`cvdata_rename --images_dir image_2 --annotations_dir label_2 --format kitti --prefix shao`
