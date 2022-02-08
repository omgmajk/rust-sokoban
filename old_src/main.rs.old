/* 
This is all compiled from:
https://sokoban.iolivia.me/c01-00-intro.html

Sokoban by iOlivia in Rust 2018
*/

// #![allow(dead_code)] // Allows for turning off warnings for dead code

// Imports / Namespaces
use ggez;
use ggez::event::{KeyCode, KeyMods};
use ggez::graphics;
use ggez::graphics::DrawParam;
use ggez::graphics::Image;
use ggez::nalgebra as na;
use ggez::{conf, event, Context, GameResult};
use specs::{
    join::Join, Builder, Component, ReadStorage, RunNow, System, VecStorage, World, WorldExt, Write, WriteStorage,
};
 
use std::path;

// Setting tile width
const TILE_WIDTH: f32 = 32.0;

// Resources
#[derive(Default)]
pub struct InputQueue {
    pub keys_pressed: Vec<KeyCode>,
}

// Registering resources
pub fn register_resources(world: &mut World) {
    world.insert(InputQueue::default())
}

// Setting up structs

// Components, procedural macros marked with #
#[derive(Debug, Component, Clone, Copy)]
#[storage(VecStorage)]
pub struct Position {
    x: u8,
    y: u8,
    z: u8, // Why do the last element end in , ?
}

#[derive(Component)]
#[storage(VecStorage)]
pub struct Renderable {
    path: String, // Again, why the , ?
}

#[derive(Component)]
#[storage(VecStorage)]
pub struct Wall {}

#[derive(Component)]
#[storage(VecStorage)]
pub struct Player {}

#[derive(Component)]
#[storage(VecStorage)]
pub struct Box {}

#[derive(Component)]
#[storage(VecStorage)]
pub struct BoxSpot {}

// Systems struct
pub struct RenderingSystem<'a> {
    context: &'a mut Context,
}

// Implementing system
impl<'a> System<'a> for RenderingSystem<'a> {
    // Data
    type SystemData = (ReadStorage<'a, Position>, ReadStorage<'a, Renderable>);

    fn run(&mut self, data: Self::SystemData) {
        let (positions, renderables) = data;

        // Clearing the screen, this gives background color
        graphics::clear(self.context, graphics::Color::new(0.95, 0.95, 0.95, 1.0));

        // Get all the renderables with their positions and sort by the position z
        let mut rendering_data = (&positions, &renderables).join().collect::<Vec<_>>();
        rendering_data.sort_by_key(|&k| k.0.z);

        // Iterate through all pairs of positions and renderables, load the image and draw it at position
        for (position, renderable) in rendering_data.iter() {
            // Load image
            let image = Image::new(self.context, renderable.path.clone()).expect("expected image");
            let x = position.x as f32 * TILE_WIDTH;
            let y = position.y as f32 * TILE_WIDTH;

            // Draw
            let draw_params = DrawParam::new().dest(na::Point2::new(x, y));
            graphics::draw(self.context, &image, draw_params).expect("expected render");
        }
        // Present content
        graphics::present(self.context).expect("expected to present");
    }
}

// This struct will hold all our game states
struct Game {
    world: World, // Again with the ,
}

// Main game loop
impl event::EventHandler for Game {
    fn update(&mut self, _context: &mut Context) -> GameResult {
        {
            let mut is = InputSystem {};
            is.run_now(&self.world);
        }
        
        Ok(())
    }

    fn draw(&mut self, context: &mut Context ) -> GameResult {
        // Render game entities
        {
            let mut rs = RenderingSystem { context };
            rs.run_now(&self.world);
        }

        Ok(())
    }
    
    fn key_down_event(
        &mut self,
        _context: &mut Context,
        keycode: KeyCode,
        _keymod: KeyMods,
        _repeat: bool,
    ) {
        println!("Key pressed {:?}", keycode);
        let mut input_queue = self.world.write_resource::<InputQueue>();
        input_queue.keys_pressed.push(keycode);
    }
}

// Implement event handler
// impl event::EventHandler for Game {
//     fn key_down_event(
//         &mut self,
//         _context: &mut Context,
//         keycode: KeyCode,
//         _keymod: KeyMods,
//         _repeat: bool,
//     ) {
//         println!("Key pressed {:?}", keycode);

//         let mut input_queue = self.world.write_resource::<InputQueue>();
//         input_queue.keys_pressed.push(keycode);
//     }
// }

// Registering components with the world
pub fn register_components(world: &mut World) {
    world.register::<Position>();
    world.register::<Renderable>();
    world.register::<Player>();
    world.register::<Wall>();
    world.register::<Box>();
    world.register::<BoxSpot>();
}

// Create a wall entity
pub fn create_wall(world: &mut World, position: Position) {
    world
        .create_entity()
        .with(Position {z: 10, ..position })
        .with(Renderable {
            path: "/images/wall.png".to_string(),
        })
        .with(Wall {})
        .build();
}

// Create a floor entity
pub fn create_floor(world: &mut World, position: Position) {
    world
        .create_entity()
        .with(Position { z: 5, ..position })
        .with(Renderable {
            path: "/images/floor.png".to_string(),
        })
        .build();
}

// Create a box entity
pub fn create_box(world: &mut World, position: Position) {
    world
        .create_entity()
        .with(Position { z: 10, ..position })
        .with(Renderable {
            path: "/images/box.png".to_string(),
        })
        .with(Box {})
        .build();
}

// Put box in spot
pub fn create_box_spot(world: &mut World, position: Position) {
    world
        .create_entity()
        .with(Position { z: 9, ..position })
        .with(Renderable {
            path: "/images/box_spot.png".to_string(),
        })
        .with(BoxSpot {})
        .build();
}

// Create a player entity
pub fn create_player(world: &mut World, position: Position) {
    world
        .create_entity()
        .with(Position { z: 10, ..position })
        .with(Renderable {
            path: "/images/player.png".to_string(),
        })
        .with(Player {})
        .build();
}

// Initialize the level, new version
pub fn initialize_level(world: &mut World) {
    const MAP: &str = "
    N N W W W W W W
    W W W . . . . W
    W . . . B . . W
    W . . . . . . W 
    W . P . . . . W
    W . . . . . . W
    W . . S . . . W
    W . . . . . . W
    W W W W W W W W
    ";

    load_map(world, MAP.to_string());
}

// Load map function
pub fn load_map(world: &mut World, map_string: String) {
    // Read all lines of map
    let rows: Vec<&str> = map_string.trim().split('\n').map(|x| x.trim()).collect();

    for (y, row) in rows.iter().enumerate() {
        let columns: Vec<&str> = row.split(' ').collect();

        for (x, column) in columns.iter().enumerate() {
            // Create the position at which to create something on the map
            let position = Position {
                x: x as u8,
                y: y as u8,
                z: 0, // We will get z from "factory" functions
            };

            // Figure out what object to create from string
            match *column {
                "." => create_floor(world, position),
                "W" => {
                    create_floor(world, position);
                    create_wall(world, position);
                } // Doesn't seem to be a need for a , after a } for some reason here?
                "P" => {
                    create_floor(world, position);
                    create_player(world, position);
                }
                "B" => {
                    create_floor(world, position);
                    create_box(world, position);
                }
                "S" => {
                    create_floor(world, position);
                    create_box_spot(world, position);
                }
                "N" => (),
                    c => panic!("Unrecognized map item {}", c), // We do end with a , however...
            }
        }
    }
}

/*
// Initialize level - This is now dead test code.
pub fn initialize_level(world: &mut World) {
    create_player(
        world,
        Position {
            x: 0,
            y: 0,
            z: 0,
        },
    );
    create_wall(
        world,
        Position {
            x: 1,
            y: 0,
            z: 0,
        },
    );
    create_box(
        world,
        Position {
            x: 2,
            y: 0,
            z: 0,
        },
    );
}
*/

// Input system
pub struct InputSystem {}

impl<'a> System<'a> for InputSystem {
    // Data
    type SystemData = (
        Write<'a, InputQueue>,
        WriteStorage<'a, Position>,
        ReadStorage<'a, Player>,
    );

    fn run(&mut self, data: Self::SystemData) {
        let (mut input_queue, mut positions, players) = data;

        for (position, _player) in (&mut positions, &players).join() {
            // Get the first key pressed
            if let Some(key) = input_queue.keys_pressed.pop() {
                // Apply the key to the position
                match key {
                    KeyCode::Up => position.y -= 1,
                    KeyCode::Down => position.y += 1,
                    KeyCode::Left => position.x -= 1,
                    KeyCode::Right => position.x += 1,
                    _ => (),
                }
            }
        }
    }
}

// fn update(&mut self, _context: &mut Context) -> GameResult {
//     // Run Input System
//     {
//         let mut is = InputSystem {};
//         is.run_now(&self.world);
//     }

//     Ok(())
// }

// Not sure why we set main to public
pub fn main() -> GameResult { // Weird way of returning shit -> before the function body. 

    let mut world = World::new();
    register_components(&mut world);
    register_resources(&mut world);
    initialize_level(&mut world);

    // Game context and event loop
    let context_builder = ggez::ContextBuilder::new("rust_sokoban", "sokoban")
    .window_setup(conf::WindowSetup::default().title("Rust Sokoban!"))
    .window_mode(conf::WindowMode::default().dimensions(800.0, 600.0))
    .add_resource_path(path::PathBuf::from("./resources"));

    let (context, event_loop) = &mut context_builder.build()?;

    // Game state
    let game = &mut Game { world };
    // Run main event loop
    event::run(context, event_loop, game) // Why are there no ; at the end of these? Because of the return above?
}