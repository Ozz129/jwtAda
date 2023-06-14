# jwtAda

// Importar las librerias que necesitas
const express = require('express');
const jwt = require('jsonwebtoken');
const cors = require('cors');

const app = express();
const port = 3000;

app.use(express.json());
app.use(cors());


// Dejar informacion quemada, simulando datos de base de datos
const USER = 'usuario';
const PSW = 'abc123';

const llaveSecreta = 'ADA-2023';

app.post('/login', (req, res) => {
    // Obtenemos la informacion, desde el body de la solicitud
    const user = req.body.usuario;
    const password = req.body.contrasenia;

    if (user === USER && password === PSW) {
        // Garantizamos la autenticacion del usuario

        // Generar el token
        const payload = {
            rol: 'contador'
        }

        const token = jwt.sign(payload, llaveSecreta)

        res.status(200).send({
            mensaje: 'Bienvenido',
            token
        })
    } else {
        // Devolvemos error y le decimos que las credenciales estan erroneas
        res.status(401).send('No autorizado')
    }
})

/**
 * {
 *  'contador': {
 *          /estadosCuenta,
 *          /registroLlegada
 * },
 * 'operario': {
 *          /registroLlegada
 * }
 *  
 * }
 */

app.get('/estadosCuenta', checkToken, checkRole(['contador']), (req, res) => {
    // Movmientos financieros
    res.status(200).send('Lista de estados de cuenta')
})

app.post('/registroLlegada', checkToken, checkRole(['operario', 'contador']), (req, res) => {
    // Control de horarios
    console.log('ROL: ', req.rol)
    res.status(200).send('Registro exitoso')
})

function checkToken(req, res, next) {
    // En esta funcion revisamos el token, que contenga el token y que el token sea valido
    const token = req.headers.authorization;

    if (!token || !token.startsWith('Bearer ')){
        // contenga token y que sea valido
        res.status(401).send({ mensage: 'El token no es valido'})
    }

    // Procedemos a validar la informacion dentro del token
    // Bearer wqetrwrterttg.sdgdsfghfdthdfyh.asdfgadsfgdsfg
    const dataToken = token.split(' ')[1];

    try {
        const decodedToken = jwt.verify(dataToken, llaveSecreta)
        req.rol = decodedToken.rol
        next();
    } catch (error) {
        res.status(401).send('Token no valido')
    }

}

/**
 * 
 * @param {*} roles me indica los roles permitidos para ese servicio ['contador']
 * @returns 
 */
function checkRole(roles) {
    //Los roles permitidos son "roles"
    return (req, res, next) => {
        const rol = req.rol
        // Valide si el rol de la solicitud esta en los roles permitidos
        if (!roles.includes(rol)) {

            // Si no esta, digale que "No estas autorizado"
            res.status(401).send('No estas autorizado')
        }

        // Si el rol si esta, continue con el proceso
        next();
    }
}


app.listen(port, () => {
    console.log('Servidor corriendo')
})
