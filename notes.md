# 21 PHYSICS
- can create own physics with mathematics and solutions like `Raycaster` 
- for realistic physics with tension, friction, bouncing, constraints, pivots, etc. it's better to use a library
## THEORY
- need to create 2 worlds: a physics world and a Three.js world 
- the physics world is purely theoretical and can't be seen, things in this world fall, collide, rub, slide etc.  
- when an object is added to the Three.js world, it also needs to be added to the physics world 
- on each frame, the physics world updates itself and we update the Three.js world accordingly 
- the hard part is to organise the code 
## LIBRARIES
- many libraries and need to decide if using 3D or 2D library
- some 3D interactions migt be reduced to 2D physics 
- 2D has better performance, good if can sum up physics to 2D collisions 
- Example: creating a pool game; balls can collide and bounce on the walls but you can project everything on a 2D plane. Can design balls as circles and the walls as simple rectangles. Won't be able to do trick shots like hitting the bottom of the ball so it can jump over other balls 
### 3D PHYSICS
- 3 main libraries
1. **Ammo.js**
    - no docs
    - direct JavaScript port of Bullet (physics engine written in C++)
    - a little heavy
    - still updated by community 
2. **Cannon.js**
    - had docs
    - lighter than Ammo
    - easier to implement than Ammo
    - mostly maintained by 1 dev
3. **Oimo.js**
    - has docs
    - lighter than Ammo
    - easier to implement than Ammo
    - moslty maintained by 1 dev
    - not been updated for 2 years
### 2D PHYSICS
- many libraries, but most popular below:
1. **Matter.js**
2. **P2.js**
3. **Planck.js**
4. **Box2D.js**
- 2D library code very similar to 3D library code, main diff is the axes you have to update 
- Ammo.js is most used lib but using Canon.js this lesson (easier to implement and understand)
## IMPORT CANNON.JS
- npm install --save cannon 
```
import CANNON from 'cannon'
```
## BASE 
### WORLD 
- create cannon.js `world`
```
const world = new CANNON.World()
```
- add gravity with the `gravity` property on that world (`Vec3`)
```
world.gravity.set(0, - 9.82, 0)
```
    - `Vec3` is vector3 for Cannon.js
    - doesn't work with https 
### OBJECT
- b/c there is a sphere in scene, need to create sphere in Cannon.js world. 
- to do this need to create a `Body`
- bodies are objects that fall and collide with other bodies 
- first need to create a shape, many available primitive shapes like `Box`, `Cylinder`, `Plane`, etc. 
- use `Sphere` w/ same radius as current sphere

```
const sphereShape = new CANNON.Sphere(0.5)
```
- create `Body` with `mass` and `position`
```
const sphereBody = new CANNON.Body({
    mass: 1,
    position: new CANNON.Vec3(0, 3, 0),
    shape: sphereShape
})
```
- add to world 
```
world.addBody(sphereBody)
```
- nothing is happening b/c need to update cannon world and three sphere accordingly 
### UPDATE CANNON.JS WORLD AND THREE.JS SCENE
- to update `World` nee to use `step(...)` function 
- 3 params:
    1. a fixed time step
    2. how much time has passed since the last step
    3. how many iterations the world can apply to catch up with a potential delay
- for time step we want experience to run at 60fps so use `1/60`
- use `3` for no. of iterations 
- for delta time need to calculate how much time has passed since the last frame 
- don't use `getDelta()` from `Clock` class
- need to subtract the `elapsedTime` from the previous frame to the current `elapsedtime`
```
const clock = new THREE.Clock()
let oldElapsedTime = 0

const tick = () =>
{
    const elapsedTime = clock.getElapsedTime()
    const deltaTime = elapsedTime - oldElapsedTime
    oldElapsedTime = elapsedTime

    // ...
}
```
- can't see anything but `sphereBody` is falling, can see this by
```
console.log(sphereBody.position.y)
```
- update Three.js `sphere` by using the `sphereBody` coordinates 
```
sphere.position.x = sphereBody.position.x
sphere.position.y = sphereBody.position.y
sphere.position.z = sphereBody.position.z
```
- better to use `copy` to copy positions for each coordinate
```
sphere.position.copy(sphereBody.position)
```
- add a new `Body` using a `Plane` shape (for the floor)
- set the `mass` to `0` so that the body is `static` 
```
const floorShape = new CANNON.Plane()
const floorBody = new CANNON.Body()
floorBody.mass = 0 
floorBody.addShape(floorShape)
world.addBody(floorBody)
```
- the sphere jumps towards the camera because the plane is facing the camera, so it needs to be rotated 
- with Cannon.js we can only use `Quaternion` and we can use the `setFromAxisAngle(...)` method 
- the first param is the rotation axis and the second param is the angle 
```
floorBody.quaternion.setFromAxisAngle(
    new CANNON.Vec3(-1, 0, 0),
    Math.PI * 0.5
)
```
## CONTACT MATERIAL 
- the ball doesn't bounce much, the friction and bouncing behaivour can be changed by setting a `Material`(diff from Three.js) and `ContactMaterial`
- `Material` is just a reference, give it a name and associate it with a `Body`
- need to create a `Material` for each type of material in your scene 
- if everything in your world is made out of plastic, only need one `Material` called **plastic** or **default** 
- if multiple materials in scene then give names like **plastic** or **concrete** 
- avoid naming things like **ball** and **ground** if you want to use same material for other objects
- create the `Material`:
```
const concreteMaterial = new CANNON.Material('concrete')
const plasticMaterial = new CANNON.Material('plastic')
```
- `ContactMaterial` is a combination of 2 `Materials` and how they should collide 
- the first 2 params are the `Materials`
- third param is an object containing collision properties like **friction** and **restitution** (how much it bounces) 
    - default value for both is `0.3`
- create `ContactMaterial` and add to the world:
```
const concretePlasticContactMaterial = new CANNON.ContactMaterial(
    concreteMaterial,
    plasticMaterial,
    {
        friction: 0.1,
        restitution: 0.7
    }
)
world.addContactMaterial(concretePlasticContactMaterial)
```
- associate the material with the bodies:
```
const sphereBody = new CANNON.Body({
    // ...
    material: plasticMaterial
})

// ...

const floorBody = new CANNON.Body()
floorBody.material = concreteMaterial
```
- simplify everything and replace the two `Materials` with a default one
- his portfolio `bruno-simon.com` only uses one material, it's all plastic b/c does not need to be realistic and is for fun 
- remove one material and rename remaining material to **default**:
```
const defaultMaterial = new CANNON.Material('default')
```
- change `ContactMaterial` name to **default**:
```
const defaultContactMaterial = new CANNON.ContactMaterial(
    defaultMaterial,
    defaultMaterial,
    {
        friction: 0.1,
        restitution: 0.7
    }
)
world.addContactMaterial(defaultContactMaterial)
```
- change sphere and floor materials to default:
```
const sphereBody = new CANNON.Body({
    // ...
    material: defaultMaterial
})

// ...

floorBody.material = defaultMaterial
```
- simplify further by removing material from sphere and floor and setting our material as the default one using the `defaultContactMaterial` property on the `World`:
```
world.defaultContactMaterial = defaultContactMaterial
```
- this way the whole world is composed of the default material which can be enough depending on how realistic your world is
## APPLY FORCES
- many ways to apply forces to a body:
### `applyForce`
- applies a force to the `Body` from a specified point in space (not necessarily on the `Body`'s surface) - examples:
    - **wind** that pushes everything a little constantly
    - small but sudden push on a domino
    - greater sudden force to make an angry bird jump toward enemy castle
### `applyImpulse`
- like `applyForce` but instead of adding to the force that results in velocity changes, it applies directly to the velocity 
### `applyLocalForce`
- same as `applyForce` but coordinates are local to the `Body` (meaning that <mark>0, 0, 0</mark> would be the center of the `Body`)
### `applyLocalImpulse`
- same as `applyImpulse` but coordinates local to the `Body`
- use `applyLocalForce(...)` to apply a small push on the `sphereBody` at the start (param 1 is `Vec3` and param 2 is the local position of the object i.e where you want the object to be pushed from):
```
sphereBody.applyLocalForce(new CANNON.Vec3(150, 0, 0), new CANNON.Vec3(0, 0, 0))
```
- mimic the wind by using `applyForce(...)` on each frame before updating the `World` and use `sphere.position` to apply the force at the right position:
```
const tick = () =>
{
    // ...

    // Update physics
    sphereBody.applyForce(new CANNON.Vec3(- 0.5, 0, 0), sphereBody.position)

    world.step(1 / 60, deltaTime, 3)

    // ...
}
```
## HANDLE MULTIPLE OBJECTS
- comment out the `sphere`, `sphereShape` and `sphereBody`
### AUTOMATE WITH FUNCTIONS 
- improve how we create spheres with a function that will add both the Three.js and Cannon.js versions 
- pass `radius` and `position` as params, but can also add other params like `mass`, `material`, `subdivisions`, etc. 
```
/**
* Utils
*/
const createSphere = (radius, position) => {
    
}
```
- create the Three.js `Mesh`:
```
const createSphere = (radius, position) =>
{
    // Three.js mesh
    const mesh = new THREE.Mesh(
        new THREE.SphereGeometry(radius, 20, 20),
        new THREE.MeshStandardMaterial({
            metalness: 0.3,
            roughness: 0.4,
            envMap: environmentMapTexture,
            envMapIntensity: 0.5
        })
    )
    mesh.castShadow = true
    mesh.position.copy(position)
    scene.add(mesh)
}
```
- create the Cannon.js `Body`:
```
const createSphere = (radius, position) =>
{
    // ...

    // Cannon.js body
    const shape = new CANNON.Sphere(radius)

    const body = new CANNON.Body({
        mass: 1,
        position: new CANNON.Vec3(0, 3, 0),
        shape: shape,
        material: defaultMaterial
    })
    body.position.copy(position)
    world.addBody(body)
}
```
- call the `createSphere(...)`:
```
createSphere(0.5, {x: 0, y: 3, z: 0})
```
- the position doesn't have to be a Three.js `Vector3` or a Cannon.js `Vec3`
- nothing is moving b/c the Three.js meshes are not updating
### USE AN ARRAY OF OBJECTS
- create array that will contain objects composed of the `Mesh` and `Body`:
```
const objectsToUpdate = []

const createSphere = (radius, position) =>
{
    // ...

    // Save in objects to update
    objectsToUpdate.push({
        mesh: mesh,
        body: body
    })
}
```
- in the `tick` function loop on that array and update the `mesh.position with the `body.position`:
```
const tick = () =>
{
    // ...

    world.step(1 / 60, deltaTime, 3)

    for(const object of objectsToUpdate)
    {
        object.mesh.position.copy(object.body.position)
    }
}
```
### ADD TO DAT.GUI
- add a `createSphere` button to Dat.GUI and create a `debugObject` to store the functions:
```
const gui = new dat.GUI()
const debugObject = {}

debugObject.createSphere = () => {
    createSphere(0.5, { x:0, y:3, z:0 })
}
gui.add(debugObject, 'createSphere')
```
- spheres are dropping in the exact same location so add randomness:
```
debugObject.createSphere = () =>
{
    createSphere(
        Math.random() * 0.5,
        {
            x: (Math.random() - 0.5) * 3,
            y: 3,
            z: (Math.random() - 0.5) * 3
        }
    )
}
```
### OPTIMISE 
- move the `sphereGeometry` and the `sphereMaterial` out of the function 
- b/c using the `radius` on the geometry, set it to `1` wen creating the geometry and update the scale of the mesh instead
```
const sphereGeometry = new THREE.SphereGeometry(1, 20, 20)
const sphereMaterial = new THREE.MeshStandardMaterial({
    metalness: 0.3,
    roughness: 0.4,
    envMap: environmentMapTexture,
    envMapIntensity: 0.5
})
const createSphere = (radius, position) =>
{
    // Three.js mesh
    const mesh = new THREE.Mesh(sphereGeometry, sphereMaterial)
    mesh.castShadow = true
    mesh.scale.set(radius, radius, radius)
    mesh.position.copy(position)
    scene.add(mesh)

    // ...
}
```
## ADD BOXES 
- do the same for boxes and use appropriate classes and params:
    - `BoxGeometry`
    - `Box`
- creating a box in `Cannon.js` is not the same as creating a box in `Three.js`
    - in Three.js you create box w/ `boxGeometry` using `width`, `height`, and `depth`
    - in Cannon.js you use `Box` and need to use `halfExtents`, it's represented by a `Vec3` corresponding to a segment that starts at the center of the box and joining one of that boxes corners
```
// Create box
const boxGeometry = new THREE.BoxGeometry(1, 1, 1)
const boxMaterial = new THREE.MeshStandardMaterial({
    metalness: 0.3,
    roughness: 0.4,
    envMap: environmentMapTexture,
    envMapIntensity: 0.5
})
const createBox = (width, height, depth, position) =>
{
    // Three.js mesh
    const mesh = new THREE.Mesh(boxGeometry, boxMaterial)
    mesh.scale.set(width, height, depth)
    mesh.castShadow = true
    mesh.position.copy(position)
    scene.add(mesh)

    // Cannon.js body
    const shape = new CANNON.Box(new CANNON.Vec3(width * 0.5, height * 0.5, depth * 0.5))

    const body = new CANNON.Body({
        mass: 1,
        position: new CANNON.Vec3(0, 3, 0),
        shape: shape,
        material: defaultMaterial
    })
    body.position.copy(position)
    world.addBody(body)

    // Save in objects
    objectsToUpdate.push({ mesh, body })
}

createBox(1, 1.5, 2, { x: 0, y: 3, z: 0 })
```
- add to `Dat.GUI`:
```
debugObject.createBox = () => {
    createBox(
        Math.random(),
        Math.random(),
        Math.random(),

         { 
            x: (Math.random() - 0.5) * 3, 
            y: 3, 
            z: (Math.random() - 0.5) * 3
        }
    )
}

// ...
gui.add(debugObject, 'createBox')
```
### ADD ROTATION TO BOXES
- Three.js `Mesh` isn't rotating like the Cannon.js `Body` is
- fix by copying the `Body` **`quaternion`** like with the `position`:
```
const tick = () =>
{
    // ...

    for(const object of objectsToUpdate)
    {
        object.mesh.position.copy(object.body.position)
        object.mesh.quaternion.copy(object.body.quaternion)
    }

    // ...
}
```
## PERFORMANCE
### BROADPHASE
- Cannon.js, when testing the collisions between objects, takes a naive approach and tests every `Body` against every other `Body`
- this is bad for performance b/c alot of work for CPU
- broadphase is doing a rough sorting of the `Bodies` before testing them and doesn't test against piles far from the object
- 3 broadphase algorithms available in Cannon.js:
    1. `NaiveBroadphase`: tests every `Bodies` against every other `Bodies`
    2. `GridBroadphase`: Quadrilles the world and only tests `Bodies` against other `Bodies` in the same grid box or the neighbors grid boxes
    3. `SAPBroadphase`(sweep and prune broadphase): tests `Bodies` on arbitrary axes during multiple steps
- default is `NaiveBroadphase` 
- recommended is `SAPBroadphase`
- using `SAPBroadphase` can eventually generate bugs where a collision doesn't occure, but this is rare and involves doing things like moving `Bodies` very fast
- to switch to `SAPBroadphase`, instantiate it in the `world.broadphase` property and use same world as param:
```
 world.broadphase = new CANNON.SAPBroadphase(world)
```
### SLEEP
- even if using an improved broadphase algorithm, all the `Bodies` are tested, even those not moving anymore
- can use a feature called sleep
- when the `Body` speed gets increadibly slow (where you can't see it moving), the `Body` can fall asleep and won't be tested unless a sufficient force is applied to it by code or if another `Body` hits it
- to activate feature, set `allowSleep` property to `true` on the `World`:
```
world.allowSleep = true
```
- **makes huge differene to performce, Bruno used on portfolio and was then able to get site to load and perform well on mobile** 
- can control how likely it is for the `Body` to fall asleep with the `sleepSpeedLimit` and `sleepTimeLimit` properties 
## EVENTS
- can listen to events on the `Body`
- useful for when you want to do things like play a sound when objects collide
- can listen to events such as `'colide'`, `'sleep'`, or `'wake up'`
- some browsers like Chrome prevent sounds from playing unless user has interacted w/ page
- play a hit sound when elements collide in JavaScript and create function to play sound:
```
/**
 * Sounds
 */
const hitSound = new Audio('/sounds/hit.mp3')

const playHitSound = () =>
{
    hitSound.play()
}
```
- listen to the `'collide'` event on the `Body` in `createBox` first:
```
/**
 * Sounds
 */
const hitSound = new Audio('/sounds/hit.mp3')

const playHitSound = () =>
{
    hitSound.play()
}
```
- add event listener
```
const createBox = (width, height, depth, position) =>
{
    // ...

    body.addEventListener('collide', playHitSound)

    // ...
}
```
- sounds weird with multiple collisions
- first problem is when calling `hitSound.play()` while sound is playing nothing happens b/c already playing
- reset sound to `0` w/ `currentTime` property:
```
const playHitSound = () =>
{
    hitSound.currentTime = 0
    hitSound.play()
}
```
- better but still hear too many hit sounds even when cube slightly touches another
- need to know how strong the impact was and not play anything if it wasn't strong enough
- to get impact strength, first need to get info about colliion by adding a param to the `collide` callback:
```
const playHitSound = (collision) =>
{
    console.log(collision)

    // ...
}
```
- impact strength can be found with `getImpactVelocityAlongNormal()` method on the `contact` property:
```
const playHitSound = (collision) =>
{
    console.log(collision.contact.getImpactVelocityAlongNormal())
    
    // ...
}
```
- only play sound if the `impactStrength` is strong enough:
```
const playHitSound = (collision) =>
{
    const impactStrength = collision.contact.getImpactVelocityAlongNormal()

    if(impactStrength > 1.5)
    {
        hitSound.currentTime = 0
        hitSound.play()
    }
}
```
- add randomness to the sound volume:
```
const playHitSound = (collision) =>
{
    const impactStrength = collision.contact.getImpactVelocityAlongNormal()

    if(impactStrength > 1.5)
    {
        hitSound.volume = Math.random()
        hitSound.currentTime = 0
        hitSound.play()
    }
}
```
- to go further could multiply different hit sounds 
- to prevent having too many sounds playing simulataneously add a short delay where sound can't play again after too soon
- can also in addition to random volume, scale it accordingly to the impact strength
- not doing in this lesson 
- copy the code used in the `createBox` function to the `createSphere` function:
```
const createSphere = (radius, position) =>
{
    // ...

    body.addEventListener('collide', playHitSound)

    // ...
}
```
## REMOVE THINGS 
- create a `reset` function and it to DAT.Gui:
```
// Reset
debugObject.reset = () =>
{
    console.log('reset')
}
gui.add(debugObject, 'reset')
```
- loop on every object inside the `objectsToUpdate` array and remove both the `object.body` from the `world` and the `object.mesh` from the `scene`, also remove the `eventListener`:
```
debugObject.reset = () =>
{
    for(const object of objectsToUpdate)
    {
        // Remove body
        object.body.removeEventListener('collide', playHitSound)
        world.removeBody(object.body)

        // Remove mesh
        scene.remove(object.mesh)
    }
}
```
## GO FURTHER WITH CANNON.JS 
### CONSTRAINTS
- enable constraints between two bodies
    1. `HingleConstraint`: acts like a door hinge
    2. `DistanceConstraint`: forces the bodies to keep a distance between each other
    3. `LockConstraint`: merges the bodies like if they were one piece
    4. `PointToPointConstraint`: glues the bodies to a specific point
### CLASSES, METHODS, PROPERTIES AND EVENTS
- many classes, each with different methods, properties and events
- browse through them to know they exists
- might be useful for future projects
### EXAMPLES
- docs aren't perfect
- spend time on demos and research to find how to do things
### WORKERS
- running the physics simulation takes time
- component of computer doing the work in CPU 
- when you run Three.js, Cannon.js, your code logic, etc. everything is done by the same thread in the CPU. 
- This thread can quickly overload if there is too much to do (like too many objects in the physics simulation), resulting in frame rate drop 
- solution is to use workers
- workers let you put a part of your code in a different tread to spread the load
- yoou can then send and recieve data from that code
- it can result in a considerable performance improvement
- problem is that the code has to be distinctly seperated
### CANNON.ES
- Cannon.js not been updated for years
- better and maintained version of Cannon.js
- Git repo : `https://github.com/pmndrs/cannon-es` 
- NPM page: `https://www.npmjs.com/package/cannon-es`
- to use this version instead of the original, remove previous cannon dependency with:
```
 npm uninstall --save cannon
 ```
 - install latest version of `Cannon.es`:
 ```
 npm install --save cannon-es
 ```
 - import:
 ```
 import * as CANNON from 'cannon-es'
 ```
 - latest version should work, if not use more specific version like `0.20`:
 ```
 npm install --save cannon-es@0.20
 ```
 ## AMMO.JS
 - we used Cannon.js b/c library easy to implement and understand
 - one of its biggest competitors is Ammo.js
 - features:
    - portage of Buller, a well known and well-oiled physics engine written in C++
    - has WebAssembly(wasm) support. WebAssemly is a low-level language supported by most recent browsers, has better performance
    - more popular and more examples of Three.js
    - supports more features
- for best performance or have particular features, best to use Ammmo.js instead of Cannon.js
## PHYSIJS
- eases implementation of physics in Three.js project
- uses Ammo.js and supports workers natively
- website: `https://chandlerprall.github.io/Physijs/`
- git repo: ` https://github.com/chandlerprall/Physijs`
- docs: `https://github.com/chandlerprall/Physijs/wiki`
- instead of creating Three.js objects and physics object, can create both simultaneously
- things can get complication when doing something not supported by library
- finding bugs can be a hassle 




