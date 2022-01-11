unify `jpeg` to `jpg` files in place:

`cvdata_convert --images_dir image_2/ --in_format jpeg --out_format jpg`


resize `kitti` images:

`cvdata_resize --input_images image_2/ --output_images resized_image_2 --input_annotations label_2/ --output_annotations resized_label_2 --width 800 --height 608 --format kitti
`
