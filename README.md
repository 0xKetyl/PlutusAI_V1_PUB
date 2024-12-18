![uiuiuiui](https://github.com/user-attachments/assets/ce5a8699-c43a-4083-b0f7-b657aa00c57b)

use async_openai::{Client, types::{CreateImageRequestArgs, ImageSize, ResponseFormat}};
use image::{ImageBuffer, Rgb};
use rand::Rng;
use tokio;

const WIDTH: u32 = 64;
const HEIGHT: u32 = 64;

struct PopcAIt {
    client: Client,
}

impl PopcAIt {
    fn new() -> Self {
        PopcAIt {
            client: Client::new(),
        }
    }

    async fn generate_avatar(&self, prompt: &str) -> ImageBuffer<Rgb<u8>, Vec<u8>> {
        let mut img = ImageBuffer::new(WIDTH, HEIGHT);
        let mut rng = rand::thread_rng();

        // Generate colors
        let bg_color = Rgb([rng.gen_range(200..256), rng.gen_range(200..256), rng.gen_range(200..256)]);
        let face_color = Rgb([rng.gen_range(150..256), rng.gen_range(100..200), rng.gen_range(100..200)]);
        let eye_color = Rgb([rng.gen_range(0..100), rng.gen_range(0..100), rng.gen_range(0..100)]);

        // Fill background
        for pixel in img.pixels_mut() {
            *pixel = bg_color;
        }

        // Draw face
        for y in 10..54 {
            for x in 10..54 {
                if (x as i32 - 32).pow(2) + (y as i32 - 32).pow(2) < 400 {
                    img.put_pixel(x, y, face_color);
                }
            }
        }

        // Draw eyes
        self.draw_eye(&mut img, 25, 30, eye_color);
        self.draw_eye(&mut img, 39, 30, eye_color);

        // Draw mouth
        for x in 28..36 {
            img.put_pixel(x, 40, Rgb([0, 0, 0]));
        }

        // Generate ChatGPT image
        let response = self.client.images().create(
            CreateImageRequestArgs::default()
                .prompt(prompt)
                .size(ImageSize::S256x256)
                .response_format(ResponseFormat::Url)
                .build().unwrap()
        ).await.unwrap();

        println!("ChatGPT generated image URL: {}", response.data[0].url.as_ref().unwrap());

        img
    }

    fn draw_eye(&self, img: &mut ImageBuffer<Rgb<u8>, Vec<u8>>, x: u32, y: u32, color: Rgb<u8>) {
        for dy in 0..5 {
            for dx in 0..5 {
                if (dx as i32 - 2).pow(2) + (dy as i32 - 2).pow(2) <= 4 {
                    img.put_pixel(x + dx, y + dy, color);
                }
            }
        }
    }
}

#[tokio::main]
async fn main() {
    let popcait = PopcAIt::new();
    let avatar = popcait.generate_avatar("A cute cat playing with a computer").await;
    avatar.save("popcait_avatar.png").unwrap();
    println!("PopcAIt avatar generated and saved as popcait_avatar.png");
}
