- 👋 Hi, I’m @BYALLL
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

<!---
BYALLL/BYALLL is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
int(current_seed)
                    p.seed = current_seed
                      
                      
                  
                p.n_iter = 1
                p.batch_size = 1
                p.do_not_save_grid = True

                if opts.img2img_color_correction:
                    p.color_corrections = initial_color_corrections
                    
                    
                wave_progress = float(1)/(float(frames_per_wave - 1))*float(((float(i)%float(frames_per_wave)) + ((float(1)/float(180))*denoising_strength_change_offset)))
                print(wave_progress)
                new_prompt = re.sub(wave_completed_regex, lambda x: set_weights(x, wave_progress), raw_prompt)
                new_prompt = re.sub(wave_remaining_regex, lambda x: set_weights(x, 1 - wave_progress), new_prompt)
                p.prompt = new_prompt
                
                print(new_prompt)

                denoising_strength_change_rate = 180/frames_per_wave

                cos = abs(math.cos(math.radians(i*denoising_strength_change_rate + denoising_strength_change_offset)))
                p.denoising_strength = initial_denoising_strength + denoising_strength_change_amplitude - (cos * denoising_strength_change_amplitude)

                state.job += f"Iteration {i + 1}/{frames}, batch {n + 1}/{batch_count}. Denoising Strength: {p.denoising_strength}"

                processed = processing.process_images(p)

                if initial_seed is None:
                    initial_seed = processed.seed
                    initial_info = processed.info

                init_img = processed.images[0]

                p.init_images = [init_img]
                
                if seed_state == "adding":
                  p.seed = processed.seed + 1
                elif seed_state == "subtracting":
                  p.seed = processed.seed - 1
                  
                image_number = int(initial_image_number + i)
                images.save_image(init_img, p.outpath_samples, "", processed.seed, processed.prompt, forced_filename=str(image_number))

                history.append(init_img)

            grid = images.image_grid(history, rows=1)
            if opts.grid_save:
                images.save_image(grid, p.outpath_grids, "grid", initial_seed, p.prompt, opts.grid_format, info=info, short_filename=not opts.grid_extended_filename, grid=True, p=p)

            grids.append(grid)
            all_images += history

        if opts.return_grid:
            all_images = grids + all_images

        if save_video:
            input_pattern = os.path.join(loopback_wave_images_path, "%d.png")
            encode_video(input_pattern, initial_image_number, loopback_wave_images_path, video_fps, video_quality, video_encoding, segment_video, video_segment_duration, ffmpeg_path)

        processed = Processed(p, all_images, initial_seed, initial_info)

        return processed
        
